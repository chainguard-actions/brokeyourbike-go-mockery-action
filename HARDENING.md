<!-- markdownlint-disable -->

# Hardening Report: brokeyourbike--go-mockery-action/v0.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **brokeyourbike--go-mockery-action/v0.2.2** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All workflow files use mutable tag or version refs instead of pinned 40-character SHA commit hashes, making the action vulnerable to supply-chain attacks if the referenced tag is moved or hijacked. Failing references include: actions/checkout@v6, actions/setup-node@v6, actions/upload-artifact@v6, codecov/codecov-action@v5, googleapis/release-please-action@v4.

Locations:

- `.github/workflows/build-test.yml:17`
- `.github/workflows/build-test.yml:20`
- `.github/workflows/check-dist.yml:24`
- `.github/workflows/check-dist.yml:27`
- `.github/workflows/check-dist.yml:44`
- `.github/workflows/coverage.yml:14`
- `.github/workflows/coverage.yml:17`
- `.github/workflows/coverage.yml:22`
- `.github/workflows/release-please.yml:14`
- `.github/workflows/release-please.yml:15`
- `.github/workflows/versions.yml:20`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` block, and no job within any workflow defines job-level `permissions:`. Without explicit permissions, workflows run with the default (potentially broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/build-test.yml:1`
- `.github/workflows/check-dist.yml:1`
- `.github/workflows/coverage.yml:1`
- `.github/workflows/release-please.yml:1`
- `.github/workflows/versions.yml:1`

### script-injection (severity: high)

Sub-rule (a): `${{ }}` expressions are interpolated directly inside `run:` shell command strings. In release-please.yml, `${{ steps.release.outputs.major }}` and `${{ steps.release.outputs.minor }}` (step outputs, which are workflow-controllable) are embedded directly in git tag and push commands — e.g. `git tag -d v${{ steps.release.outputs.major }}`. If a malicious release-please-action step sets these outputs to contain shell metacharacters, arbitrary commands could execute. These values should be passed via `env:` variables and double-quoted in the shell.

Locations:

- `.github/workflows/release-please.yml:22`

### script-injection (severity: high)

Sub-rule (a): `${{ matrix.mockery }}` is interpolated directly inside a `run:` shell command string: `./__tests__/verify-mockery.sh ${{ matrix.mockery }}`. Although the matrix values are defined statically in this workflow, any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the template engine before the shell ever sees it, bypassing shell quoting. This should be passed via an `env:` variable and double-quoted.

Locations:

- `.github/workflows/versions.yml:32`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all 5 workflow files:

1. **unpinned-uses**: Pinned all action references to full 40-char SHAs with tag comments:
   - actions/checkout@v6 → d23441a48e516b6c34aea4fa41551a30e30af803
   - actions/setup-node@v6 → 249970729cb0ef3589644e2896645e5dc5ba9c38
   - actions/upload-artifact@v6 → b7c566a772e6b6bfb58ed0dc250532a479d7789f
   - codecov/codecov-action@v5 → 0fb7174895f61a3b6b78fc075e0cd60383518dac
   - googleapis/release-please-action@v4 → 5c625bfb5d1ff62eadeeb3772007f7f66fdcf071

2. **missing-permissions**: Added `permissions: {}` at the top level of all 5 workflow files. release-please.yml job gets `contents: write` and `pull-requests: write` at the job level (needed for release-please-action and git push).

3. **script-injection (release-please.yml)**: Moved `steps.release.outputs.major`, `steps.release.outputs.minor`, and `secrets.GITHUB_TOKEN` into the step's `env:` block as `RELEASE_MAJOR`, `RELEASE_MINOR`, and `GITHUB_TOKEN`. Shell script now uses double-quoted `"${RELEASE_MAJOR}"` and `"${RELEASE_MINOR}"` throughout.

4. **script-injection (versions.yml)**: Moved `matrix.mockery` into the step's `env:` block as `MOCKERY_VERSION`. Shell script now passes `"$MOCKERY_VERSION"` (double-quoted) to verify-mockery.sh.

