# Changelog

## v1.0.0 — 2026-04-27

First public ship. Built solo.

- Six robots, four-desk pipeline (Kitty, Strategist, Engineer, Auditor)
- Model-agnostic via Rashomon LLM gateway
- Grimoire skill extraction included
- Linux x64 build

### Asset naming convention

Every release uploads two AppImage assets:

1. `Gate_X.Y.Z_amd64.AppImage` (versioned). Signed; referenced
   by the per-version URL in `scripts/updater/latest.json`.
2. `Gate-linux-x86_64.AppImage` (version-stripped permalink).
   Byte-identical to the versioned asset. Stable URL at
   `https://github.com/AkrijSama/gate-public/releases/latest/download/Gate-linux-x86_64.AppImage`.

The soliddark.net hero CTA points at the permalink URL, so the
homepage never needs editing per release. The Tauri auto-updater
continues to use the versioned URL because the manifest pins
the exact signed asset.
