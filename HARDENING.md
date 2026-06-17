<!-- markdownlint-disable -->

# Hardening Report: fabasoad--setup-enry-action/v0.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **fabasoad--setup-enry-action/v0.4.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Install enry' run: block directly interpolates the expression `${{ steps.download-enry.outputs.ref }}` inside a shell command string. This `steps.*.outputs.*` value is substituted by the GitHub Actions template engine before the shell executes the command, allowing an attacker who can influence the step output to inject arbitrary shell commands. The offending line is: `go build -ldflags="-X main.commit=$(git rev-parse HEAD) -X main.version=${{ steps.download-enry.outputs.ref }}"`

Locations:

- `action.yml:88`

### unpinned-uses (severity: high)

Three `uses:` references in action.yml use mutable tag refs instead of pinned 40-character commit SHA digests, making the action vulnerable to supply-chain attacks if those tags are moved or overwritten:
- `uses: dcarbone/install-jq-action@v3` (line 54)
- `uses: actions/checkout@v4` (line 68)
- `uses: actions/setup-go@v5` (line 77)

Locations:

- `action.yml:54`
- `action.yml:68`
- `action.yml:77`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed three unpinned action references by pinning to full commit SHAs: dcarbone/install-jq-action@v3 → @b7ef57d46ece78760b4019dbc4080a1ba2a40b45, actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5, actions/setup-go@v5 → @40f1582b2485089dde7abd97c1529aa768e1baff. Fixed script injection in the 'Install enry' step by moving `${{ steps.download-enry.outputs.ref }}` into an env: variable (ENRY_REF) and referencing it as ${ENRY_REF} in the shell command.

