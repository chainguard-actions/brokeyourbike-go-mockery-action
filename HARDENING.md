<!-- markdownlint-disable -->

# Hardening Report: brokeyourbike--go-mockery-action/v0.1.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **brokeyourbike--go-mockery-action/v0.1.7** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable version tags instead of pinned full-length SHA commit hashes. This exposes the workflow to supply-chain attacks if the referenced action tag is moved or compromised.

- build-test.yml: `actions/checkout@v3`, `actions/setup-node@v3`
- check-dist.yml: `actions/checkout@v3`, `actions/setup-node@v3`, `actions/upload-artifact@v2`
- coverage.yml: `actions/checkout@v3`, `actions/setup-node@v3`, `paambaati/codeclimate-action@v5`
- release-please.yml: `google-github-actions/release-please-action@v2`, `actions/checkout@v3`
- versions.yml: `actions/checkout@v3`

All should be pinned to a full 40-character hex SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v3`.

Locations:

- `.github/workflows/build-test.yml:16`
- `.github/workflows/build-test.yml:18`
- `.github/workflows/check-dist.yml:22`
- `.github/workflows/check-dist.yml:24`
- `.github/workflows/check-dist.yml:38`
- `.github/workflows/coverage.yml:15`
- `.github/workflows/coverage.yml:17`
- `.github/workflows/coverage.yml:21`
- `.github/workflows/release-please.yml:14`
- `.github/workflows/release-please.yml:18`
- `.github/workflows/versions.yml:20`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` block, and no job within any workflow defines its own `permissions:` block. Without explicit permissions, workflows run with the default (often broad) token permissions, violating the principle of least privilege. Each workflow should declare the minimal set of permissions required (e.g., `permissions: read-all` or specific scopes like `contents: read`).

Locations:

- `.github/workflows/build-test.yml:1`
- `.github/workflows/check-dist.yml:1`
- `.github/workflows/coverage.yml:1`
- `.github/workflows/release-please.yml:1`
- `.github/workflows/versions.yml:1`

### script-injection (severity: high)

Two workflow files interpolate `${{ ... }}` expressions directly inside `run:` shell command strings (sub-rule a), allowing template substitution to inject arbitrary shell metacharacters before the shell parses the command.

**release-please.yml** — the `tag major and patch versions` step interpolates `${{ secrets.GITHUB_TOKEN }}`, `${{ steps.release.outputs.major }}`, and `${{ steps.release.outputs.minor }}` directly into shell commands. For example:
```
git remote add gh-token "https://${{ secrets.GITHUB_TOKEN}}@github.com/..."
git tag -d v${{ steps.release.outputs.major }} || true
```
The `steps.release.outputs.*` values come from the `release-please-action` and could contain shell metacharacters. Fix by routing through env vars and double-quoting them.

**versions.yml** — the `verify mockery` step interpolates `${{ matrix.mockery }}` directly into a shell command:
```
./__tests__/verify-mockery.sh ${{ matrix.mockery }}
```
Although `matrix.mockery` is defined in the workflow itself, any `${{ ... }}` inside a `run:` block is a script-injection risk because YAML template substitution occurs before shell quoting. Fix by routing through an env var and double-quoting it.

Locations:

- `.github/workflows/release-please.yml:23`
- `.github/workflows/versions.yml:28`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all five workflow files:

**unpinned-uses**: Pinned all action references to full SHA hashes:
- actions/checkout@v3 → @a37ce9120846195fa4ece8f58b268e6043cb2f26 # v3
- actions/setup-node@v3 → @3235b876344d2a9aa001b8d1453c930bba69e610 # v3
- actions/upload-artifact@v2 → @82c141cc518b40d92cc801eee768e7aafc9c2fa2 # v2
- paambaati/codeclimate-action@v5 → @a1831d7162ea1fbc612ffe5fb3b90278b7999d59 # v5
- google-github-actions/release-please-action@v2 → @00911586051618367fa9c19b161a0a146ac51970 # v2

**missing-permissions**: Added top-level permissions blocks to all five workflows:
- build-test.yml, check-dist.yml, coverage.yml, versions.yml: `contents: read`
- release-please.yml: `contents: write` and `pull-requests: write` (needed for release-please to create PRs and push tags)

**script-injection**: Fixed both affected files:
- release-please.yml: Moved `secrets.GITHUB_TOKEN`, `steps.release.outputs.major`, and `steps.release.outputs.minor` into an `env:` block; replaced all `${{ ... }}` interpolations in the run script with `${ENV_VAR}` references
- versions.yml: Moved `matrix.mockery` into an `env:` block as `MOCKERY_VERSION`; replaced `${{ matrix.mockery }}` in the run script with `"$MOCKERY_VERSION"`

