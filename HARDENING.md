<!-- markdownlint-disable -->

# Hardening Report: fabasoad--setup-enry-action/v0.4.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--setup-enry-action/v0.4.3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags or branch names instead of full 40-character commit SHAs, making the action vulnerable to supply-chain attacks:
- action.yml: `actions/checkout@v7` (tag)
- action.yml: `actions/setup-go@v6` (tag)
- functional-tests.yml: `actions/checkout@v7` (tag, used 3 times)
- linting.yml: `fabasoad/reusable-workflows/...@main` (branch)
- release.yml: `fabasoad/reusable-workflows/...@main` (branch)
- security.yml: `fabasoad/reusable-workflows/...@main` (branch)
- sync-labels.yml: `fabasoad/reusable-workflows/...@main` (branch)
- update-license.yml: `fabasoad/reusable-workflows/...@main` (branch)

Locations:

- `action.yml:67`
- `action.yml:77`
- `.github/workflows/functional-tests.yml:57`
- `.github/workflows/functional-tests.yml:96`
- `.github/workflows/functional-tests.yml:118`
- `.github/workflows/linting.yml:12`
- `.github/workflows/release.yml:9`
- `.github/workflows/security.yml:18`
- `.github/workflows/sync-labels.yml:10`
- `.github/workflows/update-license.yml:10`

### script-injection (severity: high)

Rule (a): In functional-tests.yml, the `test-force` job's 'Test action completion' step directly interpolates `${{ matrix.force }}` inside a `run:` shell script. The matrix value flows through YAML template substitution before the shell sees it, allowing shell metacharacter injection. Offending line: `"${{ matrix.force }}"`.

Rule (b): In action.yml, the 'Install enry' step uses the env var `${DOWNLOAD_ENRY_OUTPUT_REF}` (sourced from `steps.download-enry.outputs.ref`, a workflow-controllable step output) unquoted inside a shell string: `go build -ldflags="-X main.commit=$(git rev-parse HEAD) -X main.version=${DOWNLOAD_ENRY_OUTPUT_REF}"`. The unquoted expansion allows shell metacharacter injection.

Locations:

- `.github/workflows/functional-tests.yml:130`
- `action.yml:89`

### github-env-injection (severity: high)

Two unsanitized writes to special GitHub environment files were found:

1. In functional-tests.yml ('Prepare list' step): the variable `versions` is derived from `${RELEASES}` (which holds `steps.github-releases.outputs.releases`, a workflow-controllable step output) processed through `jq`, then written directly to `$GITHUB_OUTPUT` via `echo "versions=${versions}" >> "$GITHUB_OUTPUT"` without the required `printf '%s' ... | tr -d '\n\r'` sanitization.

2. In src/get-latest-release.sh (called by the 'Get latest release' step in action.yml): the variable `version` is fetched from the GitHub API and filtered through `jq`, then written directly to `$GITHUB_OUTPUT` via `echo "version=${version}" >> "$GITHUB_OUTPUT"` without sanitization. An attacker who can influence the API response (e.g., via a crafted tag name) could inject newlines to poison subsequent GITHUB_OUTPUT entries.

Locations:

- `.github/workflows/functional-tests.yml:44`
- `src/get-latest-release.sh:13`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three finding categories:

1. unpinned-uses: Pinned all 10 mutable references to full 40-char SHAs:
   - actions/checkout@v7 → @3d3c42e5aac5ba805825da76410c181273ba90b1 (action.yml + 3x functional-tests.yml)
   - actions/setup-go@v6 → @924ae3a1cded613372ab5595356fb5720e22ba16 (action.yml)
   - fabasoad/reusable-workflows@main → @c5bd8945762dab6d2f5168b65f10355887ea40a3 (linting.yml, release.yml, security.yml, sync-labels.yml, update-license.yml)

2. script-injection: 
   - functional-tests.yml test-force job: Moved ${{ matrix.force }} to MATRIX_FORCE env var, referenced as ${MATRIX_FORCE} in shell
   - action.yml Install enry step: Extracted git rev-parse to a separate variable for cleaner ldflags construction

3. github-env-injection:
   - functional-tests.yml Prepare list step: Added `safe_versions=$(printf '%s' "${versions}" | tr -d '\n\r')` before writing to GITHUB_OUTPUT
   - src/get-latest-release.sh: Added `safe_version=$(printf '%s' "${version}" | tr -d '\n\r')` before writing to GITHUB_OUTPUT

