<!-- markdownlint-disable -->

# Hardening Report: open-policy-agent--setup-opa/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **open-policy-agent--setup-opa/v2.0.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Both workflow files reference `actions/checkout@v3`, which is a mutable tag rather than a pinned 40-character commit SHA. If the tag is moved (e.g. by a supply-chain compromise), the workflow will silently execute different code. All `uses:` references should be pinned to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v3`.

Locations:

- `.github/workflows/test_setup_opa.yml:23`
- `.github/workflows/verify_dist.yml:16`

### script-injection (severity: high)

Sub-rule (a): The `run:` block in the 'Get expected version' step directly interpolates `${{ matrix.version }}` and `${{ secrets.GITHUB_TOKEN }}` inside shell command strings. `matrix.*` values are workflow-controllable and flow through YAML template substitution before the shell processes them, allowing an attacker to inject arbitrary shell metacharacters. For example: `if [ "${{ matrix.version }}" = "<0.34" ]` and `curl --header 'authorization: Bearer ${{ secrets.GITHUB_TOKEN }}'`. These values must be passed via `env:` variables and then referenced as quoted shell variables (e.g. `"$MATRIX_VERSION"`) instead of being interpolated directly.

Locations:

- `.github/workflows/test_setup_opa.yml:30`
- `.github/workflows/test_setup_opa.yml:32`
- `.github/workflows/test_setup_opa.yml:35`
- `.github/workflows/test_setup_opa.yml:37`
- `.github/workflows/test_setup_opa.yml:40`

### github-env-injection (severity: high)

The 'Get expected version' step writes `EXPECTED_VERSION` to `$GITHUB_ENV` without sanitization. The value of `EXPECTED_VERSION` is derived from `${{ matrix.version }}` which is directly interpolated into the shell script — an attacker-controllable value. A newline character in the matrix version could inject additional key=value pairs into the GitHub environment. The write on line 41 (`echo "EXPECTED_VERSION=$EXPECTED_VERSION" >> $GITHUB_ENV`) must be preceded by sanitization: `safe=$(printf '%s' "$EXPECTED_VERSION" | tr -d '\n\r')` before writing to `$GITHUB_ENV`.

Locations:

- `.github/workflows/test_setup_opa.yml:41`

### permissions (severity: medium)

Neither workflow file defines a `permissions:` key at the top level or at the job level. Without explicit permissions, workflows inherit the repository's default token permissions, which may be overly broad (e.g. `write` access to contents, pull-requests, etc.). Both `test_setup_opa.yml` and `verify_dist.yml` should declare minimal required permissions (e.g. `permissions: contents: read`).

Locations:

- `.github/workflows/test_setup_opa.yml:1`
- `.github/workflows/verify_dist.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, permissions

**Notes:**

Fixed all four findings across both workflow files:
1. **unpinned-uses**: Pinned `actions/checkout@v3` to full SHA `f43a0e5ff2bd294095638e18286ca9a3d1956744 # v3` in both `test_setup_opa.yml` and `verify_dist.yml`.
2. **script-injection**: In the 'Get expected version' step of `test_setup_opa.yml`, moved `${{ matrix.version }}` and `${{ secrets.GITHUB_TOKEN }}` into the step's `env:` block as `MATRIX_VERSION` and `GITHUB_TOKEN`, then replaced all inline `${{ }}` expressions in the `run:` block with quoted shell variable references (`"$MATRIX_VERSION"`, `"$GITHUB_TOKEN"`).
3. **github-env-injection**: Added sanitization before writing to `$GITHUB_ENV`: `safe=$(printf '%s' "$EXPECTED_VERSION" | tr -d '\n\r')` and then writing `$safe` instead of `$EXPECTED_VERSION`.
4. **permissions**: Added `permissions: contents: read` at the top level of both workflow files, restricting the default token to the minimum needed for checkout operations.

