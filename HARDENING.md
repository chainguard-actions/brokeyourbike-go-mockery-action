<!-- markdownlint-disable -->

# Hardening Report: brokeyourbike--go-mockery-action/v0.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **brokeyourbike--go-mockery-action/v0.2.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All workflow files reference GitHub Actions using mutable version tags instead of pinned full 40-character SHA commit hashes. This exposes the pipeline to supply-chain attacks if a tag is moved or a dependency is compromised.

Failing references:
- build-test.yml: actions/checkout@v6, actions/setup-node@v6
- check-dist.yml: actions/checkout@v6, actions/setup-node@v6, actions/upload-artifact@v6
- coverage.yml: actions/checkout@v6, actions/setup-node@v6, codecov/codecov-action@v5
- release-please.yml: actions/checkout@v6, googleapis/release-please-action@v4
- versions.yml: actions/checkout@v6

Locations:

- `.github/workflows/build-test.yml:17`
- `.github/workflows/build-test.yml:19`
- `.github/workflows/check-dist.yml:22`
- `.github/workflows/check-dist.yml:24`
- `.github/workflows/check-dist.yml:38`
- `.github/workflows/coverage.yml:14`
- `.github/workflows/coverage.yml:16`
- `.github/workflows/coverage.yml:22`
- `.github/workflows/release-please.yml:13`
- `.github/workflows/release-please.yml:14`
- `.github/workflows/versions.yml:21`

### permissions (severity: medium)

None of the workflow files define a top-level `permissions:` key, and no job within any workflow defines job-level permissions. Without explicit permissions, workflows run with the default (potentially broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/build-test.yml:1`
- `.github/workflows/check-dist.yml:1`
- `.github/workflows/coverage.yml:1`
- `.github/workflows/release-please.yml:1`
- `.github/workflows/versions.yml:1`

### script-injection (severity: high)

GitHub Actions expressions are interpolated directly inside `run:` shell command strings (sub-rule a), allowing an attacker to inject arbitrary shell commands if the expression value is attacker-controlled.

1. release-please.yml: `${{ steps.release.outputs.major }}` and `${{ steps.release.outputs.minor }}` are interpolated directly in a `run:` block. These step outputs originate from the googleapis/release-please-action and could be influenced by repository content (e.g. PR titles, commit messages). Offending lines include:
   - `git remote add gh-token "https://${{ secrets.GITHUB_TOKEN}}@github.com/..."`
   - `git tag -d v${{ steps.release.outputs.major }} || true`
   - `git tag -d v${{ steps.release.outputs.major }}.${{ steps.release.outputs.minor }} || true`
   - (and several more git tag/push commands)

2. versions.yml: `${{ matrix.mockery }}` is interpolated directly in a `run:` block:
   - `./__tests__/verify-mockery.sh ${{ matrix.mockery }}`
   Matrix values are workflow-controlled and must not be interpolated directly into shell commands.

Locations:

- `.github/workflows/release-please.yml:20`
- `.github/workflows/versions.yml:29`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

Fixed all five workflow files:

**unpinned-uses**: Pinned all action references to full SHA hashes:
- actions/checkout@v6 → @df4cb1c069e1874edd31b4311f1884172cec0e10
- actions/setup-node@v6 → @249970729cb0ef3589644e2896645e5dc5ba9c38
- actions/upload-artifact@v6 → @b7c566a772e6b6bfb58ed0dc250532a479d7789f
- codecov/codecov-action@v5 → @0fb7174895f61a3b6b78fc075e0cd60383518dac
- googleapis/release-please-action@v4 → @5c625bfb5d1ff62eadeeb3772007f7f66fdcf071

**permissions**: Added top-level `permissions: {}` to all five workflow files. The release-please job gets job-level `contents: write` and `pull-requests: write` since it needs to create releases and manage PRs.

**script-injection**:
- release-please.yml: Moved GITHUB_TOKEN, RELEASE_MAJOR, and RELEASE_MINOR into the step's `env:` block; all shell commands now reference plain environment variables.
- versions.yml: Moved `${{ matrix.mockery }}` into env var MOCKERY_VERSION and referenced as `"$MOCKERY_VERSION"` in the shell script.

