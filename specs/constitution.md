# Constitution — artcraft-cli-homebrew-tap

Governing principles for every change in this repository. Amendments require a
new entry in `specs/changes/`.

## 1. Spec-driven workflow

- No implementation starts without a plan under `specs/changes/<NNN>--<slug>/plan.md`.
- Each change gets its own numbered directory; completed changes are never edited
  retroactively — corrections go into a new change directory.
- This constitution lives in `specs/constitution.md` and outranks any plan.

## 2. Generated, not hand-edited

- `Formula/*.rb` files are build artifacts produced by `script/update.py` from
  `apps.json` + a template. Never edit a formula by hand; fix the generator,
  then regenerate.
- The update script must be idempotent: re-running it on unchanged upstream
  releases produces zero diffs.

## 3. Version lockstep (fail closed)

- A formula's `version` must equal the app release version embedded in the
  release asset filename (`<app>-cli-<ver>-macos-universal.zip`), which is the
  single source of truth. The git tag (`v<ver>`) is advisory.
- If tag and asset version disagree, if the expected asset is missing, or if the
  sha256 cannot be obtained from the release's `SHA256SUMS.txt`, the script
  fails loudly for that app and leaves the existing formula untouched. It must
  never guess, fall back to partial data, or publish a broken formula.
- Checksums come only from the upstream `SHA256SUMS.txt`; the script does not
  compute checksums itself.

## 4. Integrity gates

- A change is done only when `brew audit --strict --formula Formula/<changed>.rb`
  passes and an install smoke test (`brew install ./Formula/<changed>.rb` +
  `<bin> --version`) succeeds.
- CI must reproduce the generation step and fail if committed formulas drift
  from what the generator would produce.

## 5. Upstream respect

- Only apps that ship a `macos-universal` CLI zip are tracked. New apps are
  added by appending to `apps.json` — never by special-casing in the generator.
- Apps that stop publishing the expected asset are reported, not silently
  dropped: their last-known-good formula stays pinned until a human decides.

## 6. Scope discipline

- This tap installs CLIs only. App GUIs (DMGs), Linux assets, and other
  platforms are out of scope.
- Keep formulas minimal: binary into `bin`, docs into `doc`, and **every license
  text the upstream NOTICE requires** (LICENSE-MIT, LICENSE-APACHE, NOTICE, and
  embedded-font license texts) into `doc/licenses`. No patches, no compilation,
  no bottle logic beyond the default.
- The `license` DSL must reflect the NOTICE shipped inside the release asset
  (currently "MIT or Apache-2.0, at your option" → `license any_of:`). If a
  future NOTICE changes terms, that is a human-reviewed formula/template change,
  never an automated one.

## 7. Automation safety

- CI automation may commit and push only mechanical generator output
  (formula regeneration). Any behavioral change to the generator, template,
  or `apps.json` requires a reviewed pull request.
