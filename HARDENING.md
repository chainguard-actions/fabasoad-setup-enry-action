<!-- markdownlint-disable -->

# Hardening Report: fabasoad--setup-enry-action/v0.3.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **fabasoad--setup-enry-action/v0.3.2** was hardened automatically. 12 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `${{ ... }}` expressions are interpolated directly inside `run:` shell command strings in action.yml, violating sub-rule (a). This includes:
- `${{ runner.os }}` used in `if [ "${{ runner.os }}" = "macOS" ]` and similar conditionals in the 'Collect info', 'Setup enry', and 'Clean up' steps.
- `${{ inputs.version }}` (attacker-controlled) interpolated directly into shell commands constructing filenames and echo statements in the 'Setup enry' and 'Clean up' steps.
- `${{ steps.info.outputs.ENRY_BINARY }}` and `${{ steps.info.outputs.ENRY_PATH }}` interpolated directly into shell commands.
Any `${{ ... }}` expression inside a `run:` block is a script-injection risk as the value is substituted by the template engine before the shell ever sees it, allowing injection of shell metacharacters.

Locations:

- `action.yml:17`
- `action.yml:19`
- `action.yml:37`
- `action.yml:38`
- `action.yml:40`
- `action.yml:43`
- `action.yml:44`
- `action.yml:55`
- `action.yml:60`
- `action.yml:62`

### github-env-injection (severity: high)

In the 'Setup enry' step, the value `${{ steps.info.outputs.ENRY_PATH }}` is written directly to `$GITHUB_PATH` without sanitization: `echo "${{ steps.info.outputs.ENRY_PATH }}" >> $GITHUB_PATH`. The `steps.*.outputs.*` context is an untrusted-input source per the check rules. The required sanitization step (`printf '%s' ... | tr -d '\n\r'`) is absent before the write, allowing newline injection into GITHUB_PATH.

Locations:

- `action.yml:55`

### unpinned-uses (severity: high)

The action uses `robinraju/release-downloader@v1.7`, which is pinned to a mutable version tag rather than an immutable 40-character commit SHA. If the tag is moved or the repository is compromised, a different (potentially malicious) version of the action could be executed. It should be pinned to a full SHA, e.g. `robinraju/release-downloader@<40-char-sha> # v1.7`.

Locations:

- `action.yml:28`

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

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, static-inline-injection

**Notes:**

Fixed all findings in action.yml:
1. Pinned robinraju/release-downloader@v1.7 to full SHA 768b85c8d69164800db5fc00337ab917daf3ce68 with tag comment.
2. Moved all ${{ runner.os }}, ${{ inputs.version }}, ${{ steps.info.outputs.ENRY_BINARY }}, and ${{ steps.info.outputs.ENRY_PATH }} expressions out of run: blocks into env: blocks (RUNNER_OS, INPUT_VERSION, ENRY_BINARY, ENRY_PATH env vars), referencing them as plain shell variables in the run: scripts.
3. Sanitized the ENRY_PATH value written to $GITHUB_PATH using printf '%s' "$ENRY_PATH" | tr -d '\n\r' to prevent newline injection.

