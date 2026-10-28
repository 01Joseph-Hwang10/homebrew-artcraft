# Plan: ArtCraft CLI Homebrew Tap

Tap that installs the CLI binaries of the ArtCraft "Craft Apps" from their GitHub
releases. Example install: `brew install <you>/artcraft/figcraft-cli`.

## 1. Source inventory (verified via GitHub API, 2026-10-28)

Every Craft App repo under `storytold` publishes a release tagged `v<semver>`
containing:

| Asset | Purpose |
|---|---|
| `<app>-cli-<version>-macos-universal.zip` | CLI binary, Mach-O universal (x86_64 + arm64) |
| `SHA256SUMS.txt` | Checksums for all assets (incl. the CLI zip) |

Current apps with a `-cli-…-macos-universal.zip` asset (13 total):

```
photocraft v0.16.0   pdfcraft v0.14.0    filmcraft v0.13.0   vectorcraft v0.17.0
lightcraft v0.14.0   designcraft v0.13.0 effectcraft v0.17.0 gridcraft v0.12.1
cadcraft v0.13.0     deckcraft v0.11.1   soundcraft v0.13.1  wordcraft v0.11.0
figcraft v0.11.2
```

Excluded:
- `artcraft` (main launcher) — DMG/NSIS only, no CLI asset.
- `AniCraft`, `artcraftx` — no releases yet (add formulas when they appear).

Zip layout (e.g. figcraft 0.11.2):

```
figcraft-cli-0.11.2-macos-universal/
├── figcraft-cli            # the binary (~166 MB, universal)
├── README.md, NOTICE
├── LICENSE-MIT, LICENSE-APACHE   # dual-licensed
└── OFL-*.txt               # font licenses
```

`figcraft-cli --version` → `figcraft-cli 0.11.2` (version matches the release tag
with the `v` stripped — the same holds for the asset filename, which is what the
formula should key off).

## 2. Tap structure

Repository name: `homebrew-artcraft` → tap name `artcraft`.

```
homebrew-artcraft/
├── Formula/
│   ├── deckcraft-cli.rb
│   ├── figcraft-cli.rb
│   └── … (one per app)
├── apps.json                # single source of truth: app name + repo
├── script/
│   └── update.py            # regenerates Formula/*.rb from GitHub API
└── .github/workflows/
    └── update.yml           # scheduled + manual regeneration & push
```

## 3. Formula template

Key rules:
- `version` **must equal the app release version** (tag `v0.11.2` → `version "0.11.2"`,
  matching the asset filename `<app>-cli-0.11.2-macos-universal.zip`). The zip's
  top-level dir embeds the same version, so the install block is version-aware.
- `sha256` comes from the release's `SHA256SUMS.txt` — no need to download the
  160 MB zips when generating formulas.
- macOS-only (no Linux build is shipped); universal zip covers both arches, so
  no per-arch URLs needed and no bottles (binary is already prebuilt).

```ruby
cask "deckcraft-cli" # NO — this is a formula, see below
```

```ruby
class FigcraftCli < Formula
  desc "CLI for FigCraft, the open-source design tool by ArtCraft"
  homepage "https://github.com/storytold/figcraft"
  version "0.11.2"
  license any_of: ["MIT", "Apache-2.0"]  # MIT OR Apache-2.0, per NOTICE

  # url must stay in lockstep with `version` — the asset name embeds it
  url "https://github.com/storytold/figcraft/releases/download/v#{version}/figcraft-cli-#{version}-macos-universal.zip"
  sha256 "13bad5a9464b9560036598715c7ae34d5ad0b0adcb2c67ba25798af86f774029"

  depends_on macos: :ventura # adjust to actual OS target once known

  def install
    cd "#{name}-#{version}-macos-universal" do
      bin.install "figcraft-cli"
      doc.install "README.md"
      # License compliance (see §5): ship every text the NOTICE requires
      (doc/"licenses").install "LICENSE-MIT", "LICENSE-APACHE", "NOTICE"
      (doc/"licenses/fonts").install Dir["OFL-*.txt"]  # embedded SIL OFL 1.1 fonts
      (doc/"licenses/fonts").install Dir["licenses/*"] if Dir.exist?("licenses")
    end
  end

  test do
    assert_match version.to_s, shell_output("#{bin}/figcraft-cli --version")
  end
end
```

Notes:
- Formula name = binary name = `<app>-cli` (matches the user-facing example).
- `cd "<dir>"` works because Homebrew stages the zip into a buildpath dir; use
  `within` / a glob if the dir name ever drifts.
- Drop `depends_on macos:` unless a minimum is confirmed; a plain `depends_on :macos` is safe.
- Add `livecheck` later if desired (`strategy :github_latest` on the releases page).
- **License compliance**: the NOTICE inside each release zip declares "Licensed
  under the MIT License or the Apache License, Version 2.0, at your option"
  (standard Rust dual licensing) → `license any_of: ["MIT", "Apache-2.0"]`.
  GitHub's detector on the repos shows only `Apache-2.0`, but `LICENSE-MIT` and
  `LICENSE-APACHE` both exist at every repo root; the shipped NOTICE governs the
  distributed binary. Release builds embed SIL OFL 1.1 fonts and the NOTICE
  states "each package carries their licence texts", so the install block must
  redistribute `LICENSE-MIT`, `LICENSE-APACHE`, `NOTICE`, and every `OFL-*.txt`
  (plus a repo-level `licenses/` dir if present) — dropping them would violate
  the NOTICE/OFL terms.

## 4. Version-sync & update automation

The "homebrew package version must match the app version" requirement is enforced
by generation, not hand-editing:

1. **`apps.json`** lists the apps to track:
   ```json
   { "apps": ["figcraft", "deckcraft", "photocraft", "pdfcraft", "filmcraft",
              "vectorcraft", "lightcraft", "designcraft", "effectcraft",
              "gridcraft", "cadcraft", "soundcraft", "wordcraft"] }
   ```
2. **`script/update.py`** for each app:
   - `GET /repos/storytold/<app>/releases/latest` → tag `v0.11.2`, verify asset
     `<app>-cli-0.11.2-macos-universal.zip` exists (skip app if absent).
   - Download `SHA256SUMS.txt` from the same release → extract the sha256 for the CLI zip.
   - Render the formula template → `Formula/<app>-cli.rb`.
   - `brew audit --formula` locally / in CI (stricter mode optional).
3. **CI (`.github/workflows/update.yml`)**: cron daily + `workflow_dispatch`;
   runs `script/update.py`, commits & pushes if `git diff` non-empty.
   Optionally, add a `repository_dispatch` receiver to react to release events faster.
4. **No `brew bump-*` needed** — regeneration is idempotent and self-contained.

## 5. Gotchas

- **Asset naming consistency**: pdfcraft also ships Linux CLI tarballs; the macOS
  universal zip is the only one we reference. Generation must match exactly
  `<app>-cli-<ver>-macos-universal.zip`.
- **Large binaries (~160 MB)**: no bottles; users compile nothing (it's a zip),
  but downloads are big — mention size in `desc`/README.
- **Tag vs asset version drift**: some repos could tag `v1.0-rc1` etc.; the update
  script must derive `version` from the asset filename (source of truth), not the
  tag, and fail loudly if they disagree.
- **Homebrew naming rules**: formula files must be lowercase snake_case class
  derivable from filename (`figcraft-cli` → `FigcraftCli`) — fine for all 13 names.
- **License**: dual "MIT OR Apache-2.0" per NOTICE (`license any_of:`); must
  redistribute NOTICE + both license texts + OFL font licenses (see §3 notes).
- **Tap naming**: repo must be `homebrew-<something>`; `homebrew-artcraft` gives
  `brew tap <user>/artcraft` and short installs `brew install <user>/artcraft/deckcraft-cli`.

## 6. Implementation steps

1. Create `homebrew-artcraft` repo (or init here) with structure above.
2. Write `apps.json`, formula template, `script/update.py`.
3. Run the script once → 13 formulas; `brew audit --strict --formula Formula/*.rb`
4. Smoke-test: `brew install ./Formula/figcraft-cli.rb && figcraft-cli --version`.
5. Add CI workflow, push, verify scheduled update path.
6. README with install instructions for all 13 CLIs.
