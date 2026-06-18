<!-- markdownlint-disable -->

# Hardening Report: fabasoad--setup-enry-action/v0.4.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **fabasoad--setup-enry-action/v0.4.2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions into shell command strings, violating sub-rule (a).

1. Line 88 ("Install enry" step): `go build -ldflags="-X main.commit=$(git rev-parse HEAD) -X main.version=${{ steps.download-enry.outputs.ref }}"` — the `steps.download-enry.outputs.ref` context value is interpolated directly into the shell command before the shell ever sees it, enabling shell metacharacter injection.

2. Line 97 ("Clean up" step): `run: rm -rf "${{ steps.info.outputs.bin-path }}"` — the `steps.info.outputs.bin-path` context value is interpolated directly into the shell command string, enabling path traversal or command injection if the value contains shell metacharacters.

Locations:

- `action.yml:88`
- `action.yml:97`

### unpinned-uses (severity: high)

Three `uses:` references in action.yml are pinned to mutable version tags rather than immutable 40-character commit SHAs. This exposes the action to supply-chain attacks if any of these upstream actions are compromised or their tags are moved:
- Line 54: `uses: dcarbone/install-jq-action@v3` (tag `v3`)
- Line 68: `uses: actions/checkout@v5` (tag `v5`)
- Line 77: `uses: actions/setup-go@v6` (tag `v6`)

Each should be pinned to a full SHA-1 commit hash, e.g. `actions/checkout@<40-hex-char-sha> # v5`.

Locations:

- `action.yml:54`
- `action.yml:68`
- `action.yml:77`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all three unpinned `uses:` references by pinning to full commit SHAs: dcarbone/install-jq-action@b7ef57d46ece78760b4019dbc4080a1ba2a40b45 # v3, actions/checkout@93cb6efe18208431cddfb8368fd83d5badbf9bfd # v5, actions/setup-go@4a3601121dd01d1626a1e23e37211e3254c1c06c # v6. Fixed both script injection issues: (1) In the 'Install enry' step, moved `${{ steps.download-enry.outputs.ref }}` into an `env:` block as `ENRY_REF` and referenced it as `${ENRY_REF}` in the shell command. (2) In the 'Clean up' step, moved `${{ steps.info.outputs.bin-path }}` into an `env:` block as `BIN_PATH` and referenced it as `${BIN_PATH}` in the shell command.

