---
name: shipify
description: Release a repository safely by synchronizing version files, committing, tagging, and pushing; publish GitHub Release assets for CLI and app builds on supported macOS, Linux, and Windows targets, then update channel repositories configured in ~/.shipify/setting.json through pull requests. Use when preparing or executing a software release in an existing repository.
license: MIT
metadata:
  version: "0.1.3"
---

# Shipify

Guide one repository release at a time. Inspect repository evidence first; preserve its established release contract and user changes.

## Release workflow

1. Read `~/.shipify/setting.json` and parse the channel repository paths from `channels.macos.path`, `channels.linux.path`, and `channels.windows.path`. Resolve relative paths from the repository being released. Stop and report if the file is missing, invalid, or lacks a required channel path. Then inspect repository instructions, Git status, current branch, remotes, latest tags/releases, package manifests, release workflows, and documented release commands. Identify product type, canonical version source, and required artifacts.
2. Stop if unrelated changes, an unexpected branch, missing remote, conflicting tag, or existing release makes the requested release unsafe. Do not discard or stage unrelated user changes.
3. Select requested version. Update every authoritative version file and every synchronized copy required by repository conventions. Search manifests, lockfiles, app metadata, CLI metadata, and generated version files; do not edit generated files by hand when project tooling owns them. Preserve prerelease/channel semantics.
4. After the version bump, ask the user whether the repository README should change to reflect this release. Wait for the answer. If approved, update the README before checks, commit, or push. If declined, record that the README was reviewed and left unchanged. Never commit or push before this decision and any approved README update are complete.
5. Run repository's required version, build, and release checks. Report exact failures and stop before commit when a required check fails.
6. Review complete diff. Stage only release-related paths. Commit with repository's conventions. Verify clean or understood post-commit status and confirm commit contains intended paths only.
7. Create annotated tag using repository's tag format, normally `v<version>`. Verify tag points to release commit and does not already exist.
8. Push commit and tag to intended remote. Verify remote branch and tag refs. Stop on push failure; reconcile remote state before retry.
9. If repository ships a user-installable CLI or app, inspect its release workflow and artifact contract. Build and publish GitHub Release assets for every supported macOS, Linux, and Windows target, including each documented architecture. Use platform-appropriate packages such as `.dmg`, `.pkg`, or `.zip` for macOS, `.deb`, `.rpm`, `.AppImage`, or `.tar.gz` for Linux, and `.exe`, `.msi`, or `.zip` for Windows. For a Node CLI, attach the `npm pack` tarball as a release asset. Verify filenames, checksums, signing, and notarization expectations before creating the GitHub Release with `gh release create <tag> <assets...> --verify-tag` and repository-appropriate notes. Never invent or upload unverified assets. If a supported target cannot be built or its release contract is missing, stop and report the gap. If repository is a library, skill, or source-only project without an installable CLI or app, skip binary release.
10. Assess which package channels this release needs. Resolve each repository path from `~/.shipify/setting.json`. The configured channels are owned by `systembugtj`:
   - macOS: `luban-homebrew` (Homebrew tap). Formula for a CLI executable or CLI package; cask for a macOS GUI `.app` in a DMG, ZIP, or PKG, following the repository's signing and notarization contract. If the project ships both and the tap supports both, update both; do not infer packaging from the repository name.
   - Linux: `cangjie-nix` (Nix flake). Each package is `nix/<name>.nix`, registered in `nix/packages.nix`, and listed in the README package table. Supported systems come from `flake.nix` (`x86_64-linux`, `aarch64-linux`).
   - Windows: `shennong-bucket` (Scoop bucket). One `bucket/<name>.json` manifest with `version`, `url`, `hash`, `bin`, `checkver`, and `autoupdate`. A Node CLI uses `"depends": "nodejs"`; a native binary uses a per-architecture `url` and `hash`.
   Each channel installs from the matching macOS, Linux, or Windows asset attached to this release (native archive, installer, or the `npm pack` tarball for a Node CLI), plus `SHA256SUMS`. Never use a registry URL, source archive, or `head` URL. Update all three channels when the product has a supported installable target for each OS. Skip a channel only when the product has no supported installable target for that OS. Inspect each configured checkout's remote, default branch, entries, conventions, and automation. If a checkout is missing, has no remote, or its GitHub repository does not exist, stop and report that channel; never create or publish a repository unasked.
11. Update each required channel with the same procedure. Build and verify the release artifact first; confirm its published GitHub Release asset URL is public and its checksum matches the downloaded file before editing any channel file. Update only required files, following the closest existing entry and the per-channel recipes for CLI and app in [references/channels.md](references/channels.md). Run the channel's documented check: `brew audit --strict --formula` and `brew test` (macOS), `nix build .#<name>` and `nix flake check` (needs a Linux host; if unavailable, say the Nix build was not verified), Scoop `checkver.ps1` and `autoupdate` validation (Windows; if unavailable, say so). In the channel checkout, create a release-specific branch, commit, push that branch, and open a pull request with `gh pr create`. The base is the channel's documented target branch (`develop` for `luban-homebrew`; otherwise its default branch). Describe the product type, released version, verified asset URL, checksum, and each packaging change. Treat each pull request as the release request for that channel. Never push channel changes directly to the base branch, and never fall back to another base when the target is missing: stop and ask. Report each channel as its own stage. Use `gh release` in a channel repository only if it documents a GitHub Release contract; a formula, flake, or manifest update alone does not imply one.
12. Verify final local and remote state: commit, tag, GitHub Release assets if applicable, and the state of each package channel (Homebrew, Nix, Scoop) if applicable. Summarize each completed stage and any skipped stage with reason.

## Guardrails

- Ask for missing release version or target only when repository evidence cannot determine it.
- Never force-push, overwrite a published tag/release, delete user changes, or publish unbuilt assets.
- Treat commit, tag, push, GitHub Release creation, and each channel publication (Homebrew, Nix, Scoop) as separate observable stages. Report partial success accurately.
- Prefer repository scripts and CI release workflows over hand-written build commands. Do not duplicate version or packaging logic.
- Use `gh release list`, `gh release view`, and Git remote/tag inspection to reconcile uncertain state before retrying mutations.
- A pushed tag does not prove binaries were built or attached. Verify each expected release asset explicitly.
- Do not create a GitHub Release for this skill package unless its own repository explicitly requires one.
