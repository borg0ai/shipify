# Shipify

Shipify guides repository releases: synchronize version files, commit, tag, push, publish binary GitHub Releases when the repository ships an app or CLI, and assess macOS arm64 Homebrew release needs.

## Install

```bash
npx skills add borg0ai/shipify --skill shipify --agent codex --copy --yes
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
README.md
LICENSE
```
