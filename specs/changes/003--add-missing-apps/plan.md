# Plan: Add missing apps (AniCraft, TextureCraft, MaterialCraft, Craft Anything)

Status: implemented
Depends on: 001--initial-implementation (complete), 002--readme-table-automation-and-upgrade-section (complete)

## Goal

Sync the tap with `https://getartcraft.com/apps`: the page now lists 17
CLI-shipping apps (the site's own meta says "seventeen native Crafting Apps";
the tap tracked 13 of them). Add the four missing apps:

- **AniCraft** — 2D vector animation studio
- **TextureCraft** — PBR texture painting
- **MaterialCraft** — node-graph PBR material design
- **Craft Anything** — study websites, apps, games and files with
  evidence-backed tools, then work with a coding agent to build and check your
  own project (initially mistaken for an umbrella/brand; `/apps/craft-anything`
  and `storytold/craft-anything` are canonical)

With this change the tap tracks every CLI-shipping app on the page. Per
constitution §5, all four are added by appending to `apps.json` (+ their
`descriptions` / `readme` metadata); no generator or template changes.

Non-goals: `/apps/craft` umbrella link and ArtCraft Studio (no separate
tracked repo / not on the CLI list) remain out of scope.

## Upstream verification (constitution §5 gate)

Verified 2026-05-11 via the GitHub API and each release's upstream
`SHA256SUMS.txt`:

| app | repo | latest tag | expected asset | sha256 |
|---|---|---|---|---|
| anicraft | storytold/anicraft | v0.1.0 | `anicraft-cli-0.1.0-macos-universal.zip` | `c4badaed…c80da8b` |
| texturecraft | storytold/texturecraft | v0.5.0 | `texturecraft-cli-0.5.0-macos-universal.zip` | `eb38936f…c3df8a2d` |
| materialcraft | storytold/materialcraft | v0.3.0 | `materialcraft-cli-0.3.0-macos-universal.zip` | `9854d9ef…b21e632` |
| craft-anything | storytold/craft-anything | v0.6.0 | `craft-anything-cli-0.6.0-macos-universal.zip` | `29661218…25dcb2a5` |

Zip payload spot-checks (§6 gate): every asset contains `<app>-cli`,
`README.md`, `LICENSE-MIT`, `LICENSE-APACHE`, `NOTICE`, and `OFL-*.txt` font
licenses; every NOTICE states "Licensed under the MIT License or the Apache
License, Version 2.0, at your option" → the existing `license any_of:`
template block stays correct for all four.

Schema fit: the `{app}-cli-{version}-macos-universal.zip` pattern holds with
`app = "craft-anything"`; `class_name_for("craft-anything")` →
`CraftAnythingCli` (existing Homebrew rule in `script/update.py` handles
hyphens) → formula `craft-anything-cli.rb`, binary `craft-anything-cli`. No
schema/template work.

## 1. apps.json

Append the four apps to `apps` (page order among the new three, then
craft-anything last) and add matching `descriptions` / `readme` entries in the
same style as the existing 13:

| app | desc (`descriptions`) | display_name | tagline |
|---|---|---|---|
| anicraft | CLI for AniCraft, the open-source 2D vector animation app | AniCraft | 2D vector animation |
| texturecraft | CLI for TextureCraft, the open-source PBR texture painting app | TextureCraft | PBR texture painting |
| materialcraft | CLI for MaterialCraft, the open-source PBR material design app | MaterialCraft | PBR material design |
| craft-anything | CLI for Craft Anything, the open-source software analysis and rebuilding toolkit | Craft Anything | understand & rebuild software |

## 2. Regeneration

One `script/update.py` run regenerates the 4 new formulas and the README
table; the 13 existing formulas stay byte-identical (idempotency, §2).

## 3. Integrity gates (§4) — all passed

- `brew audit --strict --formula` passes for each of the 4 new formulas.
- `brew install` + `<bin> --version` smoke tests pass: `anicraft-cli 0.1.0`,
  `texturecraft-cli` → `TextureCraft 0.5.0`, `materialcraft-cli 0.3.0`,
  `craft-anything-cli 0.6.0`; license texts + fonts shipped under
  `share/doc/<name>/licenses`.
- Idempotency re-run: all 17 formulas unchanged; `--check` drift mode exits 0.
