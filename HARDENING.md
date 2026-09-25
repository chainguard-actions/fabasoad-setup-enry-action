<!-- markdownlint-disable -->

# Hardening Report: fabasoad--setup-enry-action/v0.4.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--setup-enry-action/v0.4.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml references three actions using mutable tag refs instead of pinned 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved:
- `dcarbone/install-jq-action@v3` (line 54)
- `actions/checkout@v4` (line 60)
- `actions/setup-go@v5` (line 70)

Locations:

- `action.yml:54`
- `action.yml:60`
- `action.yml:70`

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions into shell commands, allowing template substitution before the shell ever sees the value.

1. "Install enry" step (~line 82): `go build -ldflags="-X main.commit=$(git rev-parse HEAD) -X main.version=${{ steps.download-enry.outputs.ref }}"` — `steps.download-enry.outputs.ref` is interpolated directly into the shell command string.

2. "Clean up" step (~line 91): `run: rm -rf "${{ steps.info.outputs.bin-path }}"` — `steps.info.outputs.bin-path` is interpolated directly into the shell command string. Since `bin-path` is derived from `$GITHUB_WORKSPACE` (a workflow-controlled value), this is attacker-influenced.

Locations:

- `action.yml:82`
- `action.yml:91`

### github-env-injection (severity: high)

Multiple `run:` blocks write values derived from inherited/external sources to GitHub special environment files without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`):

1. src/collect-info.sh (~line 34): `bin_path="$GITHUB_WORKSPACE/${bin_dir}"` then `echo "bin-path=${bin_path}" >> "$GITHUB_OUTPUT"` — `$GITHUB_WORKSPACE` is an inherited process env var set by the calling workflow (untrusted). It is written to GITHUB_OUTPUT without sanitization.

2. src/get-latest-release.sh (~line 13): `version=$(curl ... | jq ...)` then `echo "version=${version}" >> "$GITHUB_OUTPUT"` — `version` is fetched from an external API response and written to GITHUB_OUTPUT without sanitization.

3. action.yml "Install enry" step (~line 87): `echo "${target_dir}" >> "$GITHUB_PATH"` where `target_dir="${RUNNER_TEMP}/bin"` — `$RUNNER_TEMP` is an inherited process env var (workflow-controlled) and is written to GITHUB_PATH without sanitization.

Locations:

- `src/collect-info.sh:34`
- `src/get-latest-release.sh:13`
- `action.yml:87`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings:

1. unpinned-uses: Pinned all three action references to full 40-char commit SHAs:
   - dcarbone/install-jq-action@v3 → @b7ef57d46ece78760b4019dbc4080a1ba2a40b45 # v3
   - actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262 # v4
   - actions/setup-go@v5 → @40f1582b2485089dde7abd97c1529aa768e1baff # v5

2. script-injection: Moved ${{ }} expressions out of run: shell strings into env: blocks:
   - Install enry step: ${{ steps.download-enry.outputs.ref }} → ENRY_REF env var
   - Clean up step: ${{ steps.info.outputs.bin-path }} → BIN_PATH env var

3. github-env-injection: Added sanitization (printf '%s' ... | tr -d '\n\r') before writing to GITHUB_OUTPUT/GITHUB_PATH:
   - src/collect-info.sh: sanitized bin_path before writing to GITHUB_OUTPUT
   - src/get-latest-release.sh: sanitized version before writing to GITHUB_OUTPUT
   - action.yml Install enry step: sanitized target_dir before writing to GITHUB_PATH

