<!-- markdownlint-disable -->

# Hardening Report: fabasoad--setup-enry-action/v0.3.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **fabasoad--setup-enry-action/v0.3.3** was hardened automatically. 12 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions inside shell commands (sub-rule a). This allows template substitution to inject arbitrary shell metacharacters before the shell ever parses the command.

Affected expressions and locations:
- Line 21: `if [ "${{ runner.os }}" = "macOS" ]` (Collect info step)
- Line 23: `elif [ "${{ runner.os }}" = "Linux" ]` (Collect info step)
- Line 45: `if [ "${{ runner.os }}" = "Windows" ]` (Setup enry step)
- Line 46: `echo " enry-v${{ inputs.version }}-${{ steps.info.outputs.ENRY_BINARY }}-amd64.zip"` (Setup enry step)
- Line 47: `md5sum --check enry-v${{ inputs.version }}-${{ steps.info.outputs.ENRY_BINARY }}-amd64.zip.md5` (Setup enry step)
- Line 48: `unzip enry-v${{ inputs.version }}-${{ steps.info.outputs.ENRY_BINARY }}-amd64.zip` (Setup enry step)
- Line 50–53: multiple uses of `${{ inputs.version }}` and `${{ steps.info.outputs.ENRY_BINARY }}` (Setup enry step)
- Line 55: `echo "${{ steps.info.outputs.ENRY_PATH }}" >> $GITHUB_PATH` (Setup enry step)
- Line 61: `if [ "${{ runner.os }}" = "macOS" ]` (Clean up step)
- Line 64: `rm -f enry-v${{ inputs.version }}-${{ steps.info.outputs.ENRY_BINARY }}-amd64.*` (Clean up step)

Fix: Move all expression values into `env:` variables and reference them as double-quoted shell variables (e.g., `"$RUNNER_OS"`, `"$VERSION"`).

Locations:

- `action.yml:21`
- `action.yml:23`
- `action.yml:45`
- `action.yml:46`
- `action.yml:55`
- `action.yml:61`
- `action.yml:64`

### github-env-injection (severity: high)

In the 'Setup enry' step (line 55), the value of `${{ steps.info.outputs.ENRY_PATH }}` — a step output that is workflow-controllable — is written directly to `$GITHUB_PATH` without sanitization:

  `echo "${{ steps.info.outputs.ENRY_PATH }}" >> $GITHUB_PATH`

An attacker who can influence the step output could inject newlines to add arbitrary entries to GITHUB_PATH, potentially hijacking subsequent tool lookups. The required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`) is absent before the write.

Locations:

- `action.yml:55`

### unpinned-uses (severity: high)

The action uses `robinraju/release-downloader@v1.7` (a mutable tag reference) instead of a full 40-character commit SHA. A tag can be moved by the repository owner or a compromised account to point to malicious code, enabling a supply-chain attack.

Failing reference:
  `uses: robinraju/release-downloader@v1.7`

Fix: Pin to the exact commit SHA, e.g.:
  `uses: robinraju/release-downloader@<40-char-sha> # v1.7`

Locations:

- `action.yml:33`

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
1. Pinned robinraju/release-downloader@v1.7 to full SHA 768b85c8d69164800db5fc00337ab917daf3ce68.
2. Moved all ${{ runner.os }}, ${{ inputs.version }}, ${{ steps.info.outputs.ENRY_BINARY }}, and ${{ steps.info.outputs.ENRY_PATH }} expressions from run: blocks into env: blocks in the 'Collect info', 'Setup enry', and 'Clean up' steps. Shell scripts now reference plain env vars ($RUNNER_OS, $VERSION, $ENRY_BINARY, $ENRY_PATH).
3. Sanitized ENRY_PATH before writing to $GITHUB_PATH using printf '%s' "$ENRY_PATH" | tr -d '\n\r' to prevent newline injection.
4. Expressions remaining in with:, if:, and working-directory: fields are not shell-executed and are safe to leave as-is.

