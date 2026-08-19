<!-- markdownlint-disable -->

# Hardening Report: fabasoad--setup-enry-action/v0.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--setup-enry-action/v0.4.0** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression `${{ steps.download-enry.outputs.ref }}` is interpolated directly inside a `run:` shell command string in the 'Install enry' step. The `steps.*.outputs.*` context is workflow-controllable and flows through YAML template substitution before the shell sees it, enabling script injection. The offending line is: `go build -ldflags="-X main.commit=$(git rev-parse HEAD) -X main.version=${{ steps.download-enry.outputs.ref }}"`

Locations:

- `action.yml:88`

### unpinned-uses (severity: high)

Three `uses:` references in action.yml are pinned to mutable tags rather than full 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved: (1) `uses: dcarbone/install-jq-action@v3` (line 54), (2) `uses: actions/checkout@v4` (line 68), (3) `uses: actions/setup-go@v5` (line 77). Each should be pinned to a specific commit SHA with the tag as a comment.

Locations:

- `action.yml:54`
- `action.yml:68`
- `action.yml:77`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all three unpinned 'uses:' references by pinning to full 40-character commit SHAs with tag comments: dcarbone/install-jq-action@v3 → b7ef57d46ece78760b4019dbc4080a1ba2a40b45, actions/checkout@v4 → 11d5960a326750d5838078e36cf38b85af677262, actions/setup-go@v5 → 40f1582b2485089dde7abd97c1529aa768e1baff. Fixed script injection in the 'Install enry' step by moving ${{ steps.download-enry.outputs.ref }} out of the run: shell string into the step's env: block as ENRY_REF, then referencing it as ${ENRY_REF} in the shell command.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 5 findings across 6 files:

1. script-injection (functional-tests.yml, lines 43/75/107/143): Moved all ${{ }} expressions from run: blocks into env: blocks. RELEASES, INSTALLED, INSTALLED_1, INSTALLED_2, MATRIX_FORCE env vars now hold the values, referenced as plain shell variables.

2. script-injection (action.yml, line 107): The ENRY_REF env var (already properly in env block) is now assigned to a local _version variable with explicit double-quoting before use in the go build ldflags string.

3. github-env-injection (functional-tests.yml, line 44): The releases output is now passed via RELEASES env var using printf '%s', and the result is sanitized with tr -d '\n\r' before writing to $GITHUB_OUTPUT.

4. unpinned-uses: Pinned yakubique/github-releases@v1.2 to SHA 2827d6f627dc289b8cbbc9b4d030956d67c37c68, actions/checkout@v4 to SHA 11d5960a326750d5838078e36cf38b85af677262 (3 occurrences), and all 5 fabasoad/reusable-workflows references to SHA 10062f8186847226cb4865efbb8047795d372bae.

5. missing-permissions: Added permissions blocks to functional-tests.yml (contents: read), linting.yml (contents: read), release.yml (contents: write), sync-labels.yml (contents: read + issues: write), update-license.yml (contents: write). security.yml already had job-level permissions.

