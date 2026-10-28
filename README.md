# ArtCraft CLI Homebrew Tap

Homebrew tap for the command-line interfaces of the [ArtCraft](https://github.com/storytold) Crafting Apps.

Every formula tracks the app's upstream GitHub release **one-to-one**: the installed
CLI version always matches the app release version (e.g. `deckcraft-cli` 0.11.1 comes
from `storytold/deckcraft` tag `v0.11.1`).

## Install

```sh
brew tap 01Joseph-Hwang10/artcraft
brew install 01Joseph-Hwang10/artcraft/deckcraft-cli
```

Or install directly (tap is added automatically):

```sh
brew install 01Joseph-Hwang10/artcraft/figcraft-cli
```

## Available CLIs

| Formula | App | Latest |
|---|---|---|
| `figcraft-cli` | [FigCraft](https://github.com/storytold/figcraft) — interface design & prototyping | 0.11.2 |
| `deckcraft-cli` | [DeckCraft](https://github.com/storytold/deckcraft) — presentations & slideshows | 0.11.1 |
| `photocraft-cli` | [PhotoCraft](https://github.com/storytold/photocraft) — photo editor | 0.16.0 |
| `pdfcraft-cli` | [PdfCraft](https://github.com/storytold/pdfcraft) — PDF reader & editor | 0.14.0 |
| `filmcraft-cli` | [FilmCraft](https://github.com/storytold/filmcraft) — nonlinear video editor | 0.13.0 |
| `vectorcraft-cli` | [VectorCraft](https://github.com/storytold/vectorcraft) — vector art tool | 0.17.0 |
| `lightcraft-cli` | [LightCraft](https://github.com/storytold/lightcraft) — non-destructive photo editor | 0.14.0 |
| `designcraft-cli` | [DesignCraft](https://github.com/storytold/designcraft) — page & layout design | 0.13.0 |
| `effectcraft-cli` | [EffectCraft](https://github.com/storytold/effectcraft) — motion graphics & compositing | 0.17.0 |
| `gridcraft-cli` | [GridCraft](https://github.com/storytold/gridcraft) — spreadsheet | 0.12.1 |
| `cadcraft-cli` | [CadCraft](https://github.com/storytold/cadcraft) — CAD & drafting | 0.13.0 |
| `soundcraft-cli` | [SoundCraft](https://github.com/storytold/soundcraft) — audio workstation | 0.13.1 |
| `wordcraft-cli` | [WordCraft](https://github.com/storytold/wordcraft) — word processor | 0.11.0 |

Verify: `figcraft-cli --version` prints `figcraft-cli 0.11.2`, matching the upstream release.

## How it works

- `Formula/*.rb` are **generated artifacts** — produced by `script/update.py`
  from `apps.json` + the `storytold/*` GitHub releases. Never edit by hand.
- Versions are derived from the release **asset filename**
  (`<app>-cli-<version>-macos-universal.zip`), which is the source of truth;
  the script fails loudly if the git tag disagrees.
- Checksums are taken from each release's `SHA256SUMS.txt`.
- License compliance: each release is dual-licensed *MIT or Apache-2.0* and
  embeds SIL OFL 1.1 fonts — the formulas redistribute all required license
  texts into `doc/licenses` (see `specs/constitution.md`).
- A scheduled GitHub Action regenerates formulas daily (`workflow_dispatch`
  also available); PRs are checked for drift from generator output.

## Licensing

- This tap's own content (formula generator, CI workflow, generated formula
  files) is dual-licensed **MIT OR Apache-2.0, at your option** — see
  [`LICENSE-MIT`](LICENSE-MIT) and [`LICENSE-APACHE`](LICENSE-APACHE), mirroring
  the licensing of the apps it packages.
- The CLI binaries installed by these formulas are not covered by this
  repository's license; they keep their own upstream terms (MIT or Apache-2.0,
  plus SIL OFL 1.1 for embedded fonts), and every license text is redistributed
  into `doc/licenses`. See [`NOTICE.md`](NOTICE.md).

## A note to the ArtCraft Team

> **Hi ArtCraft maintainers!** This tap is an unofficial, community-maintained
> effort to make your CLIs one-command installable on macOS:
>
> ```sh
> brew install 01Joseph-Hwang10/artcraft/deckcraft-cli
> ```
>
> Every formula is **generated** directly from your official GitHub releases
> (version, URL, and checksum all come from your own release assets), so it
> stays in lockstep with what you publish — no forks, no patches, no mirrors.
>
> **If you're interested, I'd be happy to transfer ownership of this repository
> to the ArtCraft project** (e.g. as `storytold/homebrew-artcraft`), so it can
> become an official upstream tap — see Homebrew's
> [upstream taps](https://docs.brew.sh/How-to-Create-and-Maintain-a-Tap#upstream-taps)
> doc for why that's the recommended arrangement. A GitHub repository transfer
> preserves the history, stars, and existing installs, and old tap URLs keep
> redirecting automatically. Alternatively, I'm glad to keep maintaining it
> independently — whichever serves your users better.
>
> To get in touch: [@01Joseph-Hwang10](https://github.com/01Joseph-Hwang10) —
> open an issue here, ping me on GitHub, or just say the word and I'll transfer
> the repository to the org of your choice.

## Local development

```sh
python3 script/update.py            # regenerate all formulas
python3 script/update.py --check    # CI drift check
brew audit --strict --formula <tap>/<formula>
```

Governance: see [`specs/constitution.md`](specs/constitution.md) and
[`specs/changes/001--initial-implementation/plan.md`](specs/changes/001--initial-implementation/plan.md).
