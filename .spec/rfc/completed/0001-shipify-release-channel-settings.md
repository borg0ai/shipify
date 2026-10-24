# RFC 0001: Configure cross-platform release channels and README review

**Status:** Implemented

## Summary

Define a repeatable Shipify release flow for installable CLI and app products across supported macOS, Linux, and Windows targets. Load channel checkout locations from `~/.shipify/setting.json`, publish platform assets on the product GitHub Release, and prepare channel updates through pull requests. After each version bump, ask whether the repository README should change and finish any approved update before checks, commit, or push.

## Problem

Shipify previously described Homebrew release steps and hard-coded sibling channel locations. It did not define equivalent Nix and Scoop updates or require a README review after a version bump. This leaves channel selection dependent on local assumptions and can ship releases whose README no longer describes the product.

## Goals

- Read channel repository paths from `~/.shipify/setting.json`.
- Publish GitHub Release assets for each supported OS and architecture of installable CLI and app products.
- Update macOS, Linux, and Windows package channels through separate pull requests when the product supports those targets.
- Ask whether to update the README after the version bump, and finish approved edits before checks, commit, or push.
- Report each release stage and any skipped target with its reason.

## Non-goals

- Build new release workflows or add unsupported platform targets to a product.
- Create missing channel repositories or invent channel entries without verified release assets.
- Publish channel changes directly to channel base branches.
- Create binary releases for libraries, skills, or source-only projects.

## Design

Keep the workflow in `shipify/SKILL.md`. Read `channels.macos.path`, `channels.linux.path`, and `channels.windows.path` from `~/.shipify/setting.json`; accept absolute paths and resolve relative paths from the repository being released. Put channel-specific packaging recipes in `shipify/references/channels.md`. Update the repository `README.md` with channel and configuration details. Keep the version synchronized between `shipify/SKILL.md` frontmatter and `shipify/skill-release.json`. Stop channel work when configuration or release evidence is missing.

## Acceptance

The Skillify validator accepts the skill directory, every linked reference exists inside it, and the two version fields match. `npx skills add . --list` discovers `shipify`. The README documents the settings shape. Release instructions require supported platform assets, per-channel pull requests, and a README decision after the version bump but before commit or push. `git diff --check` passes.
