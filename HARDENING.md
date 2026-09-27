<!-- markdownlint-disable -->

# Hardening Report: fabasoad--setup-enry-action/v0.3.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--setup-enry-action/v0.3.3** was hardened automatically. 12 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The composite action uses `robinraju/release-downloader@v1.7`, which is pinned to a mutable version tag rather than an immutable 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit, enabling a supply-chain attack. It should be pinned to a full SHA, e.g. `robinraju/release-downloader@<40-char-sha> # v1.7`.

Locations:

- `action.yml:33`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions inside shell command strings (sub-rule a). Before the shell ever sees the script, GitHub Actions performs text substitution of these expressions, allowing an attacker-controlled value to inject arbitrary shell commands.

**"Collect info" step (lines 20, 22):** `${{ runner.os }}` is interpolated directly in `if [ "${{ runner.os }}" = "macOS" ]` and the `elif` branch.

**"Setup enry" step (lines 45–57):** `${{ runner.os }}`, `${{ inputs.version }}`, `${{ steps.info.outputs.ENRY_BINARY }}`, and `${{ steps.info.outputs.ENRY_PATH }}` are all interpolated directly in shell commands, including in filenames passed to `echo`, `md5sum`, `unzip`, and `tar`, and as the argument to `>> $GITHUB_PATH`.

**"Clean up" step (lines 64, 67):** `${{ runner.os }}`, `${{ inputs.version }}`, and `${{ steps.info.outputs.ENRY_BINARY }}` are interpolated directly in shell commands.

All of these should be moved to `env:` variables and referenced as quoted shell variables (e.g. `"$RUNNER_OS"`, `"$INPUT_VERSION"`) instead.

Locations:

- `action.yml:20`
- `action.yml:22`
- `action.yml:45`
- `action.yml:46`
- `action.yml:57`
- `action.yml:64`
- `action.yml:67`

### github-env-injection (severity: high)

The "Setup enry" step writes a step output value directly to `$GITHUB_PATH` without sanitization: `echo "${{ steps.info.outputs.ENRY_PATH }}" >> $GITHUB_PATH` (line 57). The value of `steps.info.outputs.ENRY_PATH` is workflow-controlled and could contain newline characters that inject additional entries into `$GITHUB_PATH`, allowing PATH hijacking. The write must be preceded by the sanitization step: `safe=$(printf '%s' "$ENRY_PATH" | tr -d '\n\r')` and then `echo "$safe" >> "$GITHUB_PATH"`.

Locations:

- `action.yml:57`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Setup enry"; move to env: map

Locations:

- `action.yml:51`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Setup enry"; move to env: map

Locations:

- `action.yml:51`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Setup enry"; move to env: map

Locations:

- `action.yml:52`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Setup enry"; move to env: map

Locations:

- `action.yml:53`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Setup enry"; move to env: map

Locations:

- `action.yml:55`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Setup enry"; move to env: map

Locations:

- `action.yml:55`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Setup enry"; move to env: map

Locations:

- `action.yml:56`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Setup enry"; move to env: map

Locations:

- `action.yml:57`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Clean up"; move to env: map

Locations:

- `action.yml:69`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all findings in hardened/action/action.yml:

1. **unpinned-uses**: Pinned `robinraju/release-downloader@v1.7` to full SHA `768b85c8d69164800db5fc00337ab917daf3ce68` with `# v1.7` comment.

2. **script-injection / static-inline-injection**: Moved all `${{ ... }}` expressions from `run:` shell strings into `env:` blocks for all three affected steps:
   - "Collect info": `runner.os` → `RUNNER_OS`
   - "Setup enry": `runner.os` → `RUNNER_OS`, `inputs.version` → `INPUT_VERSION`, `steps.info.outputs.ENRY_BINARY` → `ENRY_BINARY`, `steps.info.outputs.ENRY_PATH` → `ENRY_PATH`
   - "Clean up": `runner.os` → `RUNNER_OS`, `inputs.version` → `INPUT_VERSION`, `steps.info.outputs.ENRY_BINARY` → `ENRY_BINARY`

3. **github-env-injection**: Added `safe=$(printf '%s' "$ENRY_PATH" | tr -d '\n\r')` sanitization before writing to `$GITHUB_PATH`.

Expressions in `with:` blocks (action inputs, not shell) and `working-directory:` fields (YAML, not shell) were left as-is since they are not subject to shell injection.

