<!-- markdownlint-disable -->

# Hardening Report: fabasoad--setup-enry-action/v0.4.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--setup-enry-action/v0.4.2** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml use mutable version tags instead of pinned 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved:
- `dcarbone/install-jq-action@v3` (line 54)
- `actions/checkout@v5` (line 68)
- `actions/setup-go@v6` (line 77)
These should be replaced with full SHA digests, e.g. `actions/checkout@<40-hex-sha> # v5`.

Locations:

- `action.yml:54`
- `action.yml:68`
- `action.yml:77`

### script-injection (severity: high)

Sub-rule (a): The 'Install enry' run block directly interpolates a GitHub Actions expression inside the shell command string. Line 88 contains:
  `go build -ldflags="-X main.commit=$(git rev-parse HEAD) -X main.version=${{ steps.download-enry.outputs.ref }}"`
The `${{ steps.download-enry.outputs.ref }}` expression is expanded by the Actions template engine before the shell ever sees the command, allowing a maliciously crafted ref value to inject arbitrary shell commands. This value should be passed via an `env:` variable and then referenced as a quoted shell variable (e.g. `"$ENRY_REF"`) instead.

Locations:

- `action.yml:88`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Pinned all three unpinned `uses:` references to full 40-character commit SHAs (dcarbone/install-jq-action@b7ef57d46ece78760b4019dbc4080a1ba2a40b45, actions/checkout@fbc6f3992d24b796d5a048ff273f7fcc4a7b6c09, actions/setup-go@924ae3a1cded613372ab5595356fb5720e22ba16). Fixed script injection in the 'Install enry' step by moving `${{ steps.download-enry.outputs.ref }}` into an `env:` variable `ENRY_REF` and referencing it as `${ENRY_REF}` in the shell command.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings in action.yml:
1. Clean up step (line 98): Moved `${{ steps.info.outputs.bin-path }}` to an env var `BIN_PATH` in the step's `env:` block, then referenced it as `"${BIN_PATH}"` in the run command.
2. Install enry step (line 89): Split the single `-ldflags` argument into two separate `-ldflags` arguments so `${ENRY_REF}` is isolated in its own argument (`-ldflags "-X main.version=${ENRY_REF}"`), preventing a double-quote character in ENRY_REF from breaking out of the surrounding string context.

