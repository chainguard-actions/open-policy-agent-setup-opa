<!-- markdownlint-disable -->

# Hardening Report: open-policy-agent--setup-opa/v2.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **open-policy-agent--setup-opa/v2.4.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Both workflow files reference `actions/checkout@v6` using a mutable version tag instead of a full 40-character commit SHA. This allows the referenced action to be silently changed by a supply-chain attack. All `uses:` references must be pinned to a full SHA (e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v6`).

Failing references:
- `.github/workflows/test_setup_opa.yml` line 23: `uses: actions/checkout@v6`
- `.github/workflows/verify_dist.yml` line 19: `uses: actions/checkout@v6`

Locations:

- `.github/workflows/test_setup_opa.yml:23`
- `.github/workflows/verify_dist.yml:19`

### script-injection (severity: high)

The 'Get expected version' step in test_setup_opa.yml directly interpolates `${{ matrix.version }}` and `${{ secrets.GITHUB_TOKEN }}` inside a `run:` shell script (sub-rule a). GitHub Actions performs YAML template substitution before the shell ever sees the string, so a matrix value containing shell metacharacters (`;`, `|`, `$(...)`, etc.) would be executed by the shell. These expressions must be passed via `env:` variables and then referenced as double-quoted shell variables.

Offending lines:
- Line 32: `if [ "${{ matrix.version }}" = "<0.38" ]; then`
- Line 34: `elif [ "${{ matrix.version }}" = "0.37.1" ]; then`
- Line 37: `export EXPECTED_VERSION=$(curl --header 'authorization: Bearer ${{ secrets.GITHUB_TOKEN }}' ...)`
- Line 39: `if [ "${{ matrix.version }}" = "edge" ]; then`
- Line 42: `echo "Expected version for ${{ matrix.version }}: $EXPECTED_VERSION"`

Locations:

- `.github/workflows/test_setup_opa.yml:32`
- `.github/workflows/test_setup_opa.yml:34`
- `.github/workflows/test_setup_opa.yml:37`
- `.github/workflows/test_setup_opa.yml:39`
- `.github/workflows/test_setup_opa.yml:42`

### github-env-injection (severity: high)

The 'Get expected version' step writes `EXPECTED_VERSION` to `$GITHUB_ENV` (line 43) without sanitization. The value of `EXPECTED_VERSION` is derived from `${{ matrix.version }}` (interpolated directly into the shell script) and from the output of a `curl` call whose result is also influenced by the matrix input. A value containing newlines could inject additional environment variable assignments into `$GITHUB_ENV`. The required sanitization step (`printf '%s' "$EXPECTED_VERSION" | tr -d '\n\r'`) must be applied before the write.

Offending line:
- Line 43: `echo "EXPECTED_VERSION=$EXPECTED_VERSION" >> $GITHUB_ENV`

Locations:

- `.github/workflows/test_setup_opa.yml:43`

### missing-permissions (severity: medium)

Neither workflow file defines a top-level `permissions:` key, and no job within either file defines its own `permissions:` block. Without explicit permissions, workflows run with the repository's default token permissions, which may be overly broad (e.g. `write` access to contents). A minimal `permissions:` block should be added — for example `permissions: read-all` or specific scopes such as `contents: read`.

Affected files:
- `.github/workflows/test_setup_opa.yml` (no permissions key)
- `.github/workflows/verify_dist.yml` (no permissions key)

Locations:

- `.github/workflows/test_setup_opa.yml:1`
- `.github/workflows/verify_dist.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all four findings across both workflow files:

1. **unpinned-uses**: Pinned `actions/checkout@v6` to full SHA `df4cb1c069e1874edd31b4311f1884172cec0e10` in both `.github/workflows/test_setup_opa.yml` and `.github/workflows/verify_dist.yml`.

2. **script-injection**: In `test_setup_opa.yml`, moved `${{ matrix.version }}` and `${{ secrets.GITHUB_TOKEN }}` out of the `run:` shell script into an `env:` block (`MATRIX_VERSION` and `GITHUB_TOKEN`). All shell references now use double-quoted `"$MATRIX_VERSION"` and `"$GITHUB_TOKEN"`.

3. **github-env-injection**: Added `safe=$(printf '%s' "$EXPECTED_VERSION" | tr -d '\n\r')` before writing to `$GITHUB_ENV`, preventing newline-based injection of additional environment variables.

4. **missing-permissions**: Added `permissions: contents: read` at the top level of both workflow files, granting only the minimum permissions needed for checkout operations.

