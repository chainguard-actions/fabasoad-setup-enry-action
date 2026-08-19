<!-- markdownlint-disable -->

# Hardening Report: fabasoad--setup-enry-action/v0.3.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--setup-enry-action/v0.3.3** was hardened automatically. 14 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ ... }} expressions are interpolated directly inside run: shell command strings in action.yml.

'Collect info' step (lines 20, 22): `if [ "${{ runner.os }}" = "macOS" ]` and `elif [ "${{ runner.os }}" = "Linux" ]` — runner context injected directly into shell.

'Setup enry' step (lines 45–60): `if [ "${{ runner.os }}" = "Windows" ]`, `echo " enry-v${{ inputs.version }}-${{ steps.info.outputs.ENRY_BINARY }}-amd64.zip"`, and multiple further uses of `${{ inputs.version }}` and `${{ steps.info.outputs.ENRY_BINARY }}` in filenames passed to echo/md5sum/unzip/tar, plus `echo "${{ steps.info.outputs.ENRY_PATH }}" >> $GITHUB_PATH`.

'Clean up' step (lines 65–68): `if [ "${{ runner.os }}" = "macOS" ]` and `rm -f enry-v${{ inputs.version }}-${{ steps.info.outputs.ENRY_BINARY }}-amd64.*`.

All of these allow an attacker-controlled value (inputs.version, steps outputs, runner context) to be parsed by the shell before quoting can protect it.

Locations:

- `action.yml:20`
- `action.yml:22`
- `action.yml:45`
- `action.yml:46`
- `action.yml:60`
- `action.yml:65`
- `action.yml:68`

### script-injection (severity: high)

Sub-rule (a): ${{ ... }} expressions are interpolated directly inside run: shell command strings in pre-commit.yml.

'Update git config' step: `repo=$(echo "${{ github.repository }}" | cut -d "/" -f 2)` — github.repository is injected directly into the shell command, allowing a repository name containing shell metacharacters to break out of the echo argument.

'Run pre-commit on changed files' step: `pre-commit run --to-ref ${{ github.sha }} --from-ref origin/${{ github.base_ref }} --hook-stage=commit` and the same pattern on the next line — github.sha and github.base_ref are injected unquoted into shell arguments. github.base_ref in particular is attacker-controlled on pull_request events.

Locations:

- `.github/workflows/pre-commit.yml:24`
- `.github/workflows/pre-commit.yml:29`
- `.github/workflows/pre-commit.yml:30`

### github-env-injection (severity: high)

In the 'Setup enry' step of action.yml, the value of `${{ steps.info.outputs.ENRY_PATH }}` — a step output that is workflow-controllable — is written directly to $GITHUB_PATH without sanitization: `echo "${{ steps.info.outputs.ENRY_PATH }}" >> $GITHUB_PATH`. A newline embedded in the output value could inject arbitrary entries into GITHUB_PATH, allowing path-hijacking attacks. The required sanitization (`printf '%s' ... | tr -d '\n\r'`) is absent.

Locations:

- `action.yml:60`

### unpinned-uses (severity: high)

The following uses: references are pinned to mutable tags or version strings rather than immutable 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved:

action.yml:
  - robinraju/release-downloader@v1.7 (line 33)

.github/workflows/functional-tests.yml:
  - actions/checkout@v3 (line 24)

.github/workflows/pre-commit.yml:
  - actions/checkout@v3 (line 22)

.github/workflows/release.yml:
  - actions/checkout@v3 (line ~11)
  - simbo/changes-since-last-release-action@v1 (line ~15)
  - softprops/action-gh-release@v1 (line ~18)
  - fischerscode/tagger@v0 (line ~28)

.github/workflows/update-license.yml:
  - actions/checkout@v3 (line ~11)
  - FantasticFiasco/action-update-license-year@v3 (line ~14)

Locations:

- `action.yml:33`
- `.github/workflows/functional-tests.yml:24`
- `.github/workflows/pre-commit.yml:22`
- `.github/workflows/release.yml:11`
- `.github/workflows/release.yml:15`
- `.github/workflows/release.yml:18`
- `.github/workflows/release.yml:28`
- `.github/workflows/update-license.yml:11`
- `.github/workflows/update-license.yml:14`

### missing-permissions (severity: medium)

None of the four workflow files define a top-level `permissions:` block, and none of the individual jobs define job-level `permissions:` blocks. Without explicit permissions, workflows run with the default token permissions (which may be read-write depending on repository settings), violating the principle of least privilege.

Affected files:
  - .github/workflows/functional-tests.yml
  - .github/workflows/pre-commit.yml
  - .github/workflows/release.yml
  - .github/workflows/update-license.yml

Locations:

- `.github/workflows/functional-tests.yml:1`
- `.github/workflows/pre-commit.yml:1`
- `.github/workflows/release.yml:1`
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

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions, static-inline-injection

**Notes:**

Fixed all findings across action.yml and 4 workflow files:

1. script-injection & static-inline-injection (action.yml): Moved all ${{ runner.os }}, ${{ inputs.version }}, ${{ steps.info.outputs.ENRY_BINARY }}, and ${{ steps.info.outputs.ENRY_PATH }} expressions from run: blocks into env: blocks in 'Collect info', 'Setup enry', and 'Clean up' steps. Shell commands now reference $RUNNER_OS, $INPUT_VERSION, $ENRY_BINARY, $ENRY_PATH environment variables.

2. script-injection (pre-commit.yml): Moved ${{ github.repository }} to GITHUB_REPOSITORY env var in 'Update git config' step; moved ${{ github.sha }} and ${{ github.base_ref }} to GITHUB_SHA and GITHUB_BASE_REF env vars in 'Run pre-commit on changed files' step.

3. github-env-injection (action.yml): Added sanitization of ENRY_PATH using `printf '%s' "$ENRY_PATH" | tr -d '\n\r'` before writing to $GITHUB_PATH.

4. unpinned-uses: Pinned all 7 action references to full 40-character commit SHAs with tag comments preserved.

5. missing-permissions: Added top-level permissions blocks to all 4 workflow files with minimal required permissions (contents:read for functional-tests and pre-commit; contents:write for release; contents:write + pull-requests:write for update-license).

