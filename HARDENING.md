<!-- markdownlint-disable -->

# Hardening Report: fabasoad--setup-enry-action/v0.4.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **fabasoad--setup-enry-action/v0.4.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml use mutable version tags instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if those tags are moved:
- `dcarbone/install-jq-action@v3` (line 54)
- `actions/checkout@v4` (line 68)
- `actions/setup-go@v5` (line 77)

Locations:

- `action.yml:54`
- `action.yml:68`
- `action.yml:77`

### script-injection (severity: high)

Two `run:` blocks in action.yml directly interpolate `${{ }}` expressions into shell commands (sub-rule a). Any `${{ ... }}` expression is substituted by the Actions runner before the shell sees the string, allowing an attacker who controls the expression value to inject arbitrary shell commands.

1. Line 88 — "Install enry" step: `go build -ldflags="-X main.commit=$(git rev-parse HEAD) -X main.version=${{ steps.download-enry.outputs.ref }}"` — `steps.download-enry.outputs.ref` is a workflow-controllable context value interpolated directly into the shell command.

2. Line 97 — "Clean up" step: `run: rm -rf "${{ steps.info.outputs.bin-path }}"` — `steps.info.outputs.bin-path` is a workflow-controllable context value interpolated directly into the shell command.

Fix: move the expression values into `env:` variables and reference them as quoted shell variables (e.g., `"$INPUT_BIN_PATH"`) instead of using `${{ }}` inside `run:` blocks.

Locations:

- `action.yml:88`
- `action.yml:97`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all three unpinned uses references: dcarbone/install-jq-action@v3 → @b7ef57d46ece78760b4019dbc4080a1ba2a40b45, actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5, actions/setup-go@v5 → @40f1582b2485089dde7abd97c1529aa768e1baff. Fixed two script injection issues: (1) In 'Install enry' step, moved ${{ steps.download-enry.outputs.ref }} into env var ENRY_REF and referenced as ${ENRY_REF} in the shell command; (2) In 'Clean up' step, moved ${{ steps.info.outputs.bin-path }} into env var BIN_PATH and referenced as ${BIN_PATH} in the shell command.

