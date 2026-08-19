<!-- markdownlint-disable -->

# Hardening Report: open-policy-agent--setup-opa/v2.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **open-policy-agent--setup-opa/v2.2.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Both workflow files reference `actions/checkout@v4` using a mutable tag instead of a pinned 40-character commit SHA. This is a supply-chain risk: if the tag is moved to a different commit, malicious code could be injected into the workflow. Pin to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/test_setup_opa.yml:21`
- `.github/workflows/verify_dist.yml:16`

### missing-permissions (severity: medium)

Neither `test_setup_opa.yml` nor `verify_dist.yml` declares a top-level `permissions:` key, and no job-level `permissions:` key is present in any job. Without explicit permissions, the GITHUB_TOKEN is granted default (potentially broad write) permissions. Add a top-level `permissions: {}` block and grant only the minimum required scopes.

Locations:

- `.github/workflows/test_setup_opa.yml:1`
- `.github/workflows/verify_dist.yml:1`

### script-injection (severity: high)

The 'Get expected version' run block in test_setup_opa.yml directly interpolates `${{ matrix.version }}` and `${{ secrets.GITHUB_TOKEN }}` inside shell command strings (sub-rule a). Any `${{ ... }}` expression inside a `run:` block is subject to YAML template substitution before the shell parses it, allowing shell metacharacters to be injected. Offending lines include: `if [ "${{ matrix.version }}" = "<0.34" ]`, `curl --header 'authorization: Bearer ${{ secrets.GITHUB_TOKEN }}'`, and `echo "Expected version for ${{ matrix.version }}: $EXPECTED_VERSION"`. Move these values into `env:` variables and reference them as quoted shell variables instead.

Locations:

- `.github/workflows/test_setup_opa.yml:26`

### github-env-injection (severity: high)

The 'Get expected version' step in test_setup_opa.yml writes `echo "EXPECTED_VERSION=$EXPECTED_VERSION" >> $GITHUB_ENV` where EXPECTED_VERSION is derived from the directly-interpolated `${{ matrix.version }}` expression and from an unsanitized curl API response. No sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`) is applied before the write to $GITHUB_ENV. A newline in the value could inject arbitrary environment variables into subsequent steps. Apply sanitization before writing to $GITHUB_ENV.

Locations:

- `.github/workflows/test_setup_opa.yml:38`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings across two workflow files:

1. **unpinned-uses** (both files): Pinned `actions/checkout@v4` to full SHA `actions/checkout@11d5960a326750d5838078e36cf38b85af677262 # v4`.

2. **missing-permissions** (both files): Added `permissions: {}` at the top level of both `test_setup_opa.yml` and `verify_dist.yml`.

3. **script-injection** (test_setup_opa.yml): Moved `${{ matrix.version }}` into `MATRIX_VERSION` env var and `${{ secrets.GITHUB_TOKEN }}` into `GITHUB_TOKEN` env var in the `Get expected version` step's `env:` block. All shell code now references plain env vars.

4. **github-env-injection** (test_setup_opa.yml): Added `safe=$(printf '%s' "$EXPECTED_VERSION" | tr -d '\n\r')` before writing to `$GITHUB_ENV` to strip any embedded newlines that could inject additional environment variables.

