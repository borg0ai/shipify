<p align="center">
  <img src="logos/shipify-icon.svg" alt="Shipify package delivery icon" width="180">
</p>

# Shipify

Shipify guides repository releases: synchronize version files, commit, tag, push, publish GitHub Release assets for CLI and app builds on supported macOS, Linux, and Windows targets, then update `luban-homebrew` (macOS), `cangjie-nix` (Linux), and `shennong-bucket` (Windows) through pull requests.

Shipify reads channel checkout paths from `~/.shipify/setting.json`. Relative paths resolve from the repository being released. The file uses this shape:

```json
{
  "channels": {
    "macos": { "path": "/Volumes/ORICO/ws/prj/systembug/luban-homebrew" },
    "linux": { "path": "/Volumes/ORICO/ws/prj/systembug/cangjie-nix" },
    "windows": { "path": "/Volumes/ORICO/ws/prj/systembug/shennong-bucket" }
  }
}
```

## Install

```bash
npx skills add borg0ai/shipify --skill shipify --agent '*' --global --yes
```

## Validate and discover

Run from this repository root:

```bash
node ../skillify/skillify/scripts/validate-skill.mjs shipify
npx skills add . --list
```

## Layout

```text
shipify/
  SKILL.md
  skill-release.json
  LICENSE
  references/
    channels.md
logos/shipify-icon.svg
logos/shipify-avatar-512.png
README.md
LICENSE
```
