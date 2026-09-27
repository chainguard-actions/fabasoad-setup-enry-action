<!-- markdownlint-disable -->

# Hardening Report: fabasoad--setup-enry-action/v0.3.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--setup-enry-action/v0.3.2** was hardened automatically. 12 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The composite action step 'Download enry' uses `robinraju/release-downloader@v1.7`, which is pinned to a mutable version tag rather than an immutable 40-character commit SHA. This means the action could be silently replaced with a different (potentially malicious) version without any change to this file.

Locations:

- `action.yml:34`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions into shell command strings (sub-rule a), which causes YAML template substitution before the shell ever sees the value — allowing shell metacharacter injection.

**'Collect info' step (~line 21, 23):** `${{ runner.os }}` is interpolated directly in shell `if`/`elif` conditionals:
  `if [ "${{ runner.os }}" = "macOS" ]`
  `elif [ "${{ runner.os }}" = "Linux" ]`

**'Setup enry' step (~line 46–57):** `${{ runner.os }}`, `${{ inputs.version }}`, `${{ steps.info.outputs.ENRY_BINARY }}`, and `${{ steps.info.outputs.ENRY_PATH }}` are all interpolated directly in shell commands, including filenames passed to `echo`, `md5sum`, `unzip`, `tar`, and written to `$GITHUB_PATH`.

**'Clean up' step (~line 63, 66):** `${{ runner.os }}`, `${{ inputs.version }}`, and `${{ steps.info.outputs.ENRY_BINARY }}` are interpolated directly in shell conditionals and `rm -f` commands.

All `${{ ... }}` expressions must be moved to `env:` variables and then referenced as quoted shell variables (e.g., `"$RUNNER_OS"`) instead.

Locations:

- `action.yml:21`
- `action.yml:23`
- `action.yml:46`
- `action.yml:47`
- `action.yml:52`
- `action.yml:57`
- `action.yml:63`
- `action.yml:66`

### github-env-injection (severity: high)

In the 'Setup enry' step, the value of `${{ steps.info.outputs.ENRY_PATH }}` (a step output — a workflow-controllable, untrusted value) is written directly to `$GITHUB_PATH` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline embedded in this value could inject arbitrary entries into `$GITHUB_PATH`, enabling path-hijacking attacks.

Offending line: `echo "${{ steps.info.outputs.ENRY_PATH }}" >> $GITHUB_PATH`

The fix requires sanitizing the value before writing: `safe=$(printf '%s' "$ENRY_PATH_VAR" | tr -d '\n\r'); echo "$safe" >> "$GITHUB_PATH"`

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

Fixed all findings in action.yml:
1. Pinned robinraju/release-downloader@v1.7 to full SHA 768b85c8d69164800db5fc00337ab917daf3ce68 with # v1.7 comment.
2. Moved all ${{ runner.os }}, ${{ inputs.version }}, ${{ steps.info.outputs.ENRY_BINARY }}, and ${{ steps.info.outputs.ENRY_PATH }} expressions from run: shell strings into env: blocks in the 'Collect info', 'Setup enry', and 'Clean up' steps. Referenced them as quoted shell variables.
3. Added ENRY_PATH sanitization (printf '%s' | tr -d '\n\r') before writing to $GITHUB_PATH to prevent newline injection.
4. The ${{ inputs.version }} and ${{ steps.info.outputs.ENRY_BINARY }} in the 'Download enry' step's with: block are YAML input values (not shell commands) and were left as-is since they don't constitute shell injection.

