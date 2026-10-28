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

## Local development

```sh
python3 script/update.py            # regenerate all formulas
python3 script/update.py --check    # CI drift check
brew audit --strict --formula <tap>/<formula>
```

Governance: see [`specs/constitution.md`](specs/constitution.md) and
[`specs/changes/001--initial-implementation/plan.md`](specs/changes/001--initial-implementation/plan.md).
