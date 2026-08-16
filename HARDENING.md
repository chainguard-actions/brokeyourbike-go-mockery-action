<!-- markdownlint-disable -->

# Hardening Report: brokeyourbike--go-mockery-action/v0.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **brokeyourbike--go-mockery-action/v0.2.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across all workflow files use mutable version tags instead of immutable 40-character SHA commit hashes, making the workflows vulnerable to supply-chain attacks if a tag is moved. Failing references include: actions/checkout@v6, actions/setup-node@v6, actions/upload-artifact@v6, codecov/codecov-action@v5, googleapis/release-please-action@v4.

Locations:

- `.github/workflows/build-test.yml:18`
- `.github/workflows/build-test.yml:20`
- `.github/workflows/check-dist.yml:23`
- `.github/workflows/check-dist.yml:25`
- `.github/workflows/check-dist.yml:44`
- `.github/workflows/coverage.yml:14`
- `.github/workflows/coverage.yml:16`
- `.github/workflows/coverage.yml:20`
- `.github/workflows/release-please.yml:14`
- `.github/workflows/release-please.yml:15`
- `.github/workflows/versions.yml:20`

### permissions (severity: medium)

None of the workflow files define a top-level `permissions:` block, and no job within any workflow defines job-level permissions. Without explicit permissions, workflows run with the default (potentially broad) token permissions. All 5 workflow files are affected.

Locations:

- `.github/workflows/build-test.yml:1`
- `.github/workflows/check-dist.yml:1`
- `.github/workflows/coverage.yml:1`
- `.github/workflows/release-please.yml:1`
- `.github/workflows/versions.yml:1`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are interpolated directly inside `run:` shell command strings, allowing injection of shell metacharacters before the shell ever sees the value.

(1) `.github/workflows/versions.yml` line 28: `${{ matrix.mockery }}` is passed directly as a shell argument: `./__tests__/verify-mockery.sh ${{ matrix.mockery }}`. The `matrix.*` context is workflow-controllable.

(2) `.github/workflows/release-please.yml` lines 22–31: `${{ steps.release.outputs.major }}` and `${{ steps.release.outputs.minor }}` are interpolated directly into multiple `git tag` and `git push` shell commands. The `steps.*.outputs.*` context is workflow-controllable and must be passed via an `env:` block with double-quoted shell expansion instead.

Locations:

- `.github/workflows/versions.yml:28`
- `.github/workflows/release-please.yml:22`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

Fixed all 3 findings across 5 workflow files:

1. unpinned-uses: Pinned all 5 action references to full 40-char SHAs with tag comments preserved: actions/checkout@d23441a48e516b6c34aea4fa41551a30e30af803 # v6, actions/setup-node@249970729cb0ef3589644e2896645e5dc5ba9c38 # v6, actions/upload-artifact@b7c566a772e6b6bfb58ed0dc250532a479d7789f # v6, codecov/codecov-action@0fb7174895f61a3b6b78fc075e0cd60383518dac # v5, googleapis/release-please-action@5c625bfb5d1ff62eadeeb3772007f7f66fdcf071 # v4.

2. permissions: Added top-level permissions blocks to all 5 workflows. build-test.yml, check-dist.yml, coverage.yml, and versions.yml get 'contents: read'. release-please.yml gets 'contents: write' and 'pull-requests: write' (minimum needed for release-please to create PRs and push tags).

3. script-injection: (a) versions.yml: moved ${{ matrix.mockery }} into env.MOCKERY_VERSION and referenced as "$MOCKERY_VERSION" in the shell run block. (b) release-please.yml: moved ${{ steps.release.outputs.major }}, ${{ steps.release.outputs.minor }}, and ${{ secrets.GITHUB_TOKEN }} into env block as RELEASE_MAJOR, RELEASE_MINOR, GITHUB_TOKEN and referenced as shell variables throughout the git tag/push commands.

