<!-- markdownlint-disable -->

# Hardening Report: open-policy-agent--setup-opa/v2.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **open-policy-agent--setup-opa/v2.3.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Both workflow files reference `actions/checkout@v5`, which is a mutable tag rather than a pinned 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit at any time, enabling supply-chain attacks. All `uses:` references should be pinned to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v5`.

Locations:

- `.github/workflows/test_setup_opa.yml:23`
- `.github/workflows/verify_dist.yml:17`

### missing-permissions (severity: medium)

Neither workflow file declares a top-level `permissions:` key, and no job in either file has its own `permissions:` block. Without explicit permissions, workflows run with the default (potentially broad) token permissions. A minimal `permissions:` block (e.g. `contents: read`) should be added at the top level of each workflow.

Locations:

- `.github/workflows/test_setup_opa.yml:1`
- `.github/workflows/verify_dist.yml:1`

### script-injection (severity: high)

Multiple `${{ ... }}` expressions are interpolated directly inside `run:` shell command strings in the 'Get expected version' step. Specifically, `${{ matrix.version }}` is used in shell `if` comparisons and an `echo` statement, and `${{ secrets.GITHUB_TOKEN }}` is embedded in a `curl` command. These values flow through YAML template substitution before the shell parses them, meaning a specially crafted matrix value (e.g. from a fork PR) could inject arbitrary shell commands. Rule (a) violation. Fix: move these values into `env:` variables and reference them as quoted shell variables (e.g. `"$MATRIX_VERSION"`).

Locations:

- `.github/workflows/test_setup_opa.yml:31`
- `.github/workflows/test_setup_opa.yml:33`
- `.github/workflows/test_setup_opa.yml:36`
- `.github/workflows/test_setup_opa.yml:39`
- `.github/workflows/test_setup_opa.yml:42`

### github-env-injection (severity: high)

The 'Get expected version' step writes `EXPECTED_VERSION` to `$GITHUB_ENV` without sanitization. `EXPECTED_VERSION` is derived from `${{ matrix.version }}` (a workflow-controllable value) and from the GitHub API response (`jq -r .name`). Neither source is sanitized with `printf '%s' ... | tr -d '\n\r'` before the write (`echo "EXPECTED_VERSION=$EXPECTED_VERSION" >> $GITHUB_ENV`). A newline character in the value could inject additional environment variables into subsequent steps. Fix: sanitize the value before writing, e.g. `safe=$(printf '%s' "$EXPECTED_VERSION" | tr -d '\n\r'); echo "EXPECTED_VERSION=$safe" >> "$GITHUB_ENV"`.

Locations:

- `.github/workflows/test_setup_opa.yml:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings across both workflow files:
1. unpinned-uses: Pinned `actions/checkout@v5` to full SHA `fbc6f3992d24b796d5a048ff273f7fcc4a7b6c09` in both .github/workflows/test_setup_opa.yml and .github/workflows/verify_dist.yml, preserving the tag in a comment.
2. missing-permissions: Added `permissions: contents: read` at the top level of both workflow files.
3. script-injection: In test_setup_opa.yml's 'Get expected version' step, moved `${{ matrix.version }}` into `MATRIX_VERSION` env var and `${{ secrets.GITHUB_TOKEN }}` into `GITHUB_TOKEN` env var; all shell references now use the plain env var names (`$MATRIX_VERSION`, `$GITHUB_TOKEN`).
4. github-env-injection: Added sanitization before writing to $GITHUB_ENV: `safe=$(printf '%s' "$EXPECTED_VERSION" | tr -d '\n\r')` and then writing `$safe` instead of the raw value.

