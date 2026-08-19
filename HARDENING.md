<!-- markdownlint-disable -->

# Hardening Report: fabasoad--setup-enry-action/v0.3.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--setup-enry-action/v0.3.2** was hardened automatically. 13 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references across workflow files and action.yml use mutable tags instead of full 40-character SHA commit digests, making the action vulnerable to supply-chain attacks if the referenced tag is moved or overwritten.

Failing references:
- `.github/workflows/create-release.yml`: `actions/checkout@v3`, `simbo/changes-since-last-release-action@v1`, `softprops/action-gh-release@v1`, `fischerscode/tagger@v0`
- `.github/workflows/functional-tests.yml`: `actions/checkout@v3`
- `.github/workflows/update-license.yml`: `actions/checkout@v3`, `FantasticFiasco/action-update-license-year@v3`
- `action.yml`: `robinraju/release-downloader@v1.7`

Locations:

- `.github/workflows/create-release.yml:13`
- `.github/workflows/create-release.yml:16`
- `.github/workflows/create-release.yml:19`
- `.github/workflows/create-release.yml:27`
- `.github/workflows/functional-tests.yml:22`
- `.github/workflows/update-license.yml:10`
- `.github/workflows/update-license.yml:11`
- `action.yml:30`

### script-injection (severity: high)

Multiple `run:` blocks in `action.yml` directly interpolate GitHub Actions expressions inside shell command strings (sub-rule a). These expressions are substituted by the Actions runner before the shell parses the command, allowing an attacker who controls the input to inject arbitrary shell commands.

- **"Collect info" step** (line ~17): `${{ runner.os }}` is interpolated directly in `if [ "${{ runner.os }}" = "macOS" ]` and similar comparisons inside the shell script.
- **"Setup enry" step** (line ~44): `${{ runner.os }}`, `${{ inputs.version }}`, and `${{ steps.info.outputs.ENRY_BINARY }}` are interpolated directly in shell commands such as `echo " enry-v${{ inputs.version }}-${{ steps.info.outputs.ENRY_BINARY }}-amd64.zip" >> ...` and `unzip enry-v${{ inputs.version }}-...`.
- **"Clean up" step** (line ~60): `${{ runner.os }}`, `${{ inputs.version }}`, and `${{ steps.info.outputs.ENRY_BINARY }}` are interpolated directly in `rm -f enry-v${{ inputs.version }}-${{ steps.info.outputs.ENRY_BINARY }}-amd64.*`.

All `${{ ... }}` expressions must be moved to `env:` variables and then referenced as double-quoted shell variables (e.g., `"$RUNNER_OS"`, `"$INPUT_VERSION"`).

Locations:

- `action.yml:17`
- `action.yml:44`
- `action.yml:60`

### github-env-injection (severity: high)

The **"Setup enry"** step writes `${{ steps.info.outputs.ENRY_PATH }}` directly to `$GITHUB_PATH` without sanitization:

```sh
echo "${{ steps.info.outputs.ENRY_PATH }}" >> $GITHUB_PATH
```

The value of `steps.info.outputs.ENRY_PATH` is a workflow-controlled value (a step output). Writing it to `$GITHUB_PATH` without first stripping newlines via `printf '%s' "$VALUE" | tr -d '\n\r'` allows an attacker to inject additional PATH entries by embedding newline characters in the output. The fix is to sanitize the value before writing:

```sh
safe=$(printf '%s' "${{ steps.info.outputs.ENRY_PATH }}" | tr -d '\n\r')
echo "$safe" >> "$GITHUB_PATH"
```

Locations:

- `action.yml:57`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` key, and no job within any of them defines a job-level `permissions:` key. Without explicit permissions, workflows run with the default token permissions, which may be overly broad (e.g., `write` access to contents and pull requests). Each workflow should declare the minimal required permissions.

- `create-release.yml`: needs at minimum `contents: write` for creating releases.
- `functional-tests.yml`: needs at minimum `contents: read`.
- `update-license.yml`: needs at minimum `contents: write` and `pull-requests: write`.

Locations:

- `.github/workflows/create-release.yml:1`
- `.github/workflows/functional-tests.yml:1`
- `.github/workflows/update-license.yml:1`

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

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions, static-inline-injection

**Notes:**

Fixed all findings across action.yml and the three workflow files:

1. **unpinned-uses**: Pinned all 6 action references to full 40-char SHAs with tag comments preserved.

2. **script-injection / static-inline-injection**: In action.yml, moved all ${{ runner.os }}, ${{ inputs.version }}, ${{ steps.info.outputs.ENRY_BINARY }}, and ${{ steps.info.outputs.ENRY_PATH }} expressions out of run: shell blocks into step-level env: maps. Shell scripts now reference plain env vars ($RUNNER_OS, $INPUT_VERSION, $ENRY_BINARY, $ENRY_PATH).

3. **github-env-injection**: In the 'Setup enry' step, ENRY_PATH is now sanitized via `safe=$(printf '%s' "$ENRY_PATH" | tr -d '\n\r')` before being written to $GITHUB_PATH.

4. **missing-permissions**: Added top-level permissions blocks — `contents: write` for create-release.yml, `contents: read` for functional-tests.yml, and `contents: write` + `pull-requests: write` for update-license.yml.

