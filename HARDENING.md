<!-- markdownlint-disable -->

# Hardening Report: brokeyourbike--go-mockery-action/v0.2.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **brokeyourbike--go-mockery-action/v0.2.3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable version tags instead of pinned 40-character SHA commits, making them vulnerable to supply-chain attacks if the tag is moved. Failing references include: actions/checkout@v6, actions/setup-node@v6, actions/upload-artifact@v6, codecov/codecov-action@v5, googleapis/release-please-action@v4.

Locations:

- `.github/workflows/build-test.yml:17`
- `.github/workflows/build-test.yml:19`
- `.github/workflows/check-dist.yml:22`
- `.github/workflows/check-dist.yml:24`
- `.github/workflows/check-dist.yml:38`
- `.github/workflows/coverage.yml:15`
- `.github/workflows/coverage.yml:17`
- `.github/workflows/coverage.yml:21`
- `.github/workflows/dependabot.yml:18`
- `.github/workflows/dependabot.yml:20`
- `.github/workflows/release-please.yml:12`
- `.github/workflows/release-please.yml:13`
- `.github/workflows/versions.yml:20`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are directly interpolated inside run: shell command strings. In release-please.yml, `${{ secrets.GITHUB_TOKEN }}`, `${{ steps.release.outputs.major }}`, and `${{ steps.release.outputs.minor }}` are embedded directly in shell commands (e.g., `git remote add gh-token "https://${{ secrets.GITHUB_TOKEN}}@github.com/..."`, `git tag -d v${{ steps.release.outputs.major }}`). In versions.yml, `${{ matrix.mockery }}` is passed as an unquoted argument to a shell script (`./__tests__/verify-mockery.sh ${{ matrix.mockery }}`). All ${{ }} expressions must be moved to env: vars and then double-quoted in the shell.

Locations:

- `.github/workflows/release-please.yml:21`
- `.github/workflows/versions.yml:27`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, workflows run with the repository's default token permissions (which may be overly broad). Each file should declare minimal required permissions.

Locations:

- `.github/workflows/build-test.yml:1`
- `.github/workflows/check-dist.yml:1`
- `.github/workflows/coverage.yml:1`
- `.github/workflows/release-please.yml:1`
- `.github/workflows/versions.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all findings across 6 workflow files:

1. unpinned-uses: Pinned all action references to full 40-char SHAs with tag comments preserved:
   - actions/checkout@v6 → d23441a48e516b6c34aea4fa41551a30e30af803
   - actions/setup-node@v6 → 249970729cb0ef3589644e2896645e5dc5ba9c38
   - actions/upload-artifact@v6 → b7c566a772e6b6bfb58ed0dc250532a479d7789f
   - codecov/codecov-action@v5 → 0fb7174895f61a3b6b78fc075e0cd60383518dac
   - googleapis/release-please-action@v4 → 5c625bfb5d1ff62eadeeb3772007f7f66fdcf071

2. script-injection: Moved all ${{ }} expressions out of run: shell strings into env: blocks:
   - release-please.yml: GITHUB_TOKEN, RELEASE_MAJOR, RELEASE_MINOR env vars
   - versions.yml: MOCKERY_VERSION env var, referenced as "$MOCKERY_VERSION"

3. missing-permissions: Added top-level permissions blocks to all 5 affected workflows:
   - build-test.yml, check-dist.yml, coverage.yml, versions.yml: contents: read
   - release-please.yml: contents: write + pull-requests: write (required for release-please)
   - dependabot.yml already had contents: write (preserved)

