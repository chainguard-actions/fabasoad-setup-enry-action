<!-- markdownlint-disable -->

# Hardening Report: fabasoad--setup-enry-action/v0.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--setup-enry-action/v0.4.0** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Install enry' run: block in action.yml directly interpolates a ${{ steps.download-enry.outputs.ref }} expression inside the shell command string: `go build -ldflags="-X main.commit=$(git rev-parse HEAD) -X main.version=${{ steps.download-enry.outputs.ref }}"`. The steps.*.outputs.* context flows through YAML template substitution before the shell processes it, allowing an attacker who can influence that output value to inject arbitrary shell commands.

Locations:

- `action.yml:88`

### unpinned-uses (severity: high)

action.yml references three external actions using mutable tag refs instead of pinned 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved: `dcarbone/install-jq-action@v3` (line 54), `actions/checkout@v4` (line 68), `actions/setup-go@v5` (line 77).

Locations:

- `action.yml:54`
- `action.yml:68`
- `action.yml:77`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all three unpinned action references by pinning to full commit SHAs: dcarbone/install-jq-action@v3 → b7ef57d46ece78760b4019dbc4080a1ba2a40b45, actions/checkout@v4 → 11d5960a326750d5838078e36cf38b85af677262, actions/setup-go@v5 → 40f1582b2485089dde7abd97c1529aa768e1baff. Fixed script injection in the 'Install enry' step by moving ${{ steps.download-enry.outputs.ref }} into the env: block as ENRY_VERSION and referencing it as ${ENRY_VERSION} in the shell command.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed src/collect-info.sh: added sanitization of bin_path before writing to $GITHUB_OUTPUT. The $GITHUB_WORKSPACE environment variable is workflow-controlled and could contain newline characters that would inject arbitrary key=value pairs into $GITHUB_OUTPUT. Added `safe_bin_path=$(printf '%s' "$bin_path" | tr -d '\n\r')` and changed the echo to use `safe_bin_path` instead of `bin_path`. The script uses POSIX sh (#!/usr/bin/env sh), so the fix uses only POSIX-compatible commands.

