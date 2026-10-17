---
name: shipify
description: Release a repository safely by synchronizing version files, committing, tagging, and pushing; publish GitHub Release assets for apps or CLI binaries and assess macOS arm64 Homebrew releases. Use when preparing or executing a software release in an existing repository.
license: MIT
metadata:
  version: "0.1.1"
---

# Shipify

Guide one repository release at a time. Inspect repository evidence first; preserve its established release contract and user changes.

## Release workflow

1. Inspect repository instructions, Git status, current branch, remotes, latest tags/releases, package manifests, release workflows, and documented release commands. Identify product type, canonical version source, and required artifacts.
2. Stop if unrelated changes, an unexpected branch, missing remote, conflicting tag, or existing release makes the requested release unsafe. Do not discard or stage unrelated user changes.
3. Select requested version. Update every authoritative version file and every synchronized copy required by repository conventions. Search manifests, lockfiles, app metadata, CLI metadata, and generated version files; do not edit generated files by hand when project tooling owns them. Preserve prerelease/channel semantics.
4. Run repository's required version, build, and release checks. Report exact failures and stop before commit when a required check fails.
5. Review complete diff. Stage only release-related paths. Commit with repository's conventions. Verify clean or understood post-commit status and confirm commit contains intended paths only.
6. Create annotated tag using repository's tag format, normally `v<version>`. Verify tag points to release commit and does not already exist.
7. Push commit and tag to intended remote. Verify remote branch and tag refs. Stop on push failure; reconcile remote state before retry.
8. Determine whether repository ships a user-installable app or CLI binary. If yes, inspect its release workflow and artifact contract. Build the documented target assets, verify filenames/checksums/signing expectations, then create GitHub Release with `gh release create <tag> <assets...> --verify-tag` and repository-appropriate notes. Never invent or upload unverified assets. If repository is a library, skill, or source-only project without binary release contract, skip GitHub binary release.
9. Assess whether this release also needs a Homebrew tap update. The tap repository is `https://github.com/systembugtj/luban-homebrew`. Inspect its default branch, existing formulae and casks, supported architecture, and release automation. Classify the installable release artifacts: CLI executable or CLI package uses a formula; macOS GUI app packaged as an installable `.app` in a DMG, ZIP, or PKG uses a cask, following the repository's signing and notarization contract. If the project ships both and the tap supports both, update both entries; do not infer packaging from the repository name alone. Skip Homebrew changes for libraries, skills, source-only projects, or products without a supported macOS artifact. Build and verify each required artifact first; confirm its published asset URL and checksum before editing the matching formula or cask. In the tap checkout, create a release-specific branch, update only required formula or cask files, run the tap's documented checks, commit, and push that branch. Create a pull request against the tap's default branch with `gh pr create`; describe the product type, released version, verified asset URL, checksum, and reason for each packaging change. Treat this pull request as the release request. Never push the tap change directly to its default branch. Stop and report if the tap checkout, GitHub authentication, or required release evidence is unavailable. Use `gh release` only if `luban-homebrew` itself documents a GitHub Release asset/tag contract; a Homebrew formula or cask update alone does not imply a GitHub Release.
10. Verify final local and remote state: commit, tag, GitHub Release assets if applicable, and Homebrew formula/tap state if applicable. Summarize each completed stage and any skipped stage with reason.

## Guardrails

- Ask for missing release version or target only when repository evidence cannot determine it.
- Never force-push, overwrite a published tag/release, delete user changes, or publish unbuilt assets.
- Treat commit, tag, push, GitHub Release creation, and Homebrew publication as separate observable stages. Report partial success accurately.
- Prefer repository scripts and CI release workflows over hand-written build commands. Do not duplicate version or packaging logic.
- Use `gh release list`, `gh release view`, and Git remote/tag inspection to reconcile uncertain state before retrying mutations.
- A pushed tag does not prove binaries were built or attached. Verify each expected release asset explicitly.
- Do not create a GitHub Release for this skill package unless its own repository explicitly requires one.
