<!-- markdownlint-disable -->

# Hardening Report: open-policy-agent--setup-opa/v2.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **open-policy-agent--setup-opa/v2.1.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Both workflow files reference `actions/checkout@v3` using a mutable tag instead of a pinned 40-character commit SHA. This exposes the workflow to supply-chain attacks if the tag is moved to a malicious commit. Failing references: `actions/checkout@v3` in both .github/workflows/test_setup_opa.yml and .github/workflows/verify_dist.yml.

Locations:

- `.github/workflows/test_setup_opa.yml:21`
- `.github/workflows/verify_dist.yml:16`

### missing-permissions (severity: medium)

Neither workflow file defines a top-level `permissions:` block, and no job in either file defines job-level permissions. Without explicit permissions, the GITHUB_TOKEN is granted default (potentially write) permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/test_setup_opa.yml:1`
- `.github/workflows/verify_dist.yml:1`

### script-injection (severity: high)

Sub-rule (a): The 'Get expected version' run: block in test_setup_opa.yml directly interpolates `${{ matrix.version }}` and `${{ secrets.GITHUB_TOKEN }}` inside shell command strings. `matrix.version` is a workflow-controllable value that flows through YAML template substitution before the shell parses it, enabling script injection. Offending lines include: `if [ "${{ matrix.version }}" = "<0.34" ]`, `elif [ "${{ matrix.version }}" = "0.34.1" ]`, `if [ "${{ matrix.version }}" = "edge" ]`, `echo "Expected version for ${{ matrix.version }}: $EXPECTED_VERSION"`, and `authorization: Bearer ${{ secrets.GITHUB_TOKEN }}`.

Locations:

- `.github/workflows/test_setup_opa.yml:29`

### github-env-injection (severity: high)

The 'Get expected version' step writes `EXPECTED_VERSION` to `$GITHUB_ENV` without sanitization. The value of `EXPECTED_VERSION` is derived from `${{ matrix.version }}` (a workflow-controllable matrix value) interpolated directly into the shell script. A newline character in the value could inject arbitrary environment variables into subsequent steps. The write `echo "EXPECTED_VERSION=$EXPECTED_VERSION" >> $GITHUB_ENV` is missing the required `printf '%s' ... | tr -d '\n\r'` sanitization step.

Locations:

- `.github/workflows/test_setup_opa.yml:40`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings across both workflow files:

1. **unpinned-uses** (both files): Pinned `actions/checkout@v3` to full SHA `a37ce9120846195fa4ece8f58b268e6043cb2f26` with `# v3` comment for readability.

2. **missing-permissions** (both files): Added `permissions: contents: read` top-level block to both workflows. `contents: read` is the minimum needed for `actions/checkout` to work.

3. **script-injection** (test_setup_opa.yml): Moved `${{ matrix.version }}` into `MATRIX_VERSION` env var and `${{ secrets.GITHUB_TOKEN }}` into `GH_TOKEN` env var on the 'Get expected version' step. All shell code now references plain environment variables instead of inline `${{ }}` expressions.

4. **github-env-injection** (test_setup_opa.yml): Added `safe=$(printf '%s' "$EXPECTED_VERSION" | tr -d '\n\r')` sanitization step before writing to `$GITHUB_ENV`, preventing newline-based environment variable injection.

