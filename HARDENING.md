<!-- markdownlint-disable -->

# Hardening Report: fabasoad--setup-enry-action/v0.4.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--setup-enry-action/v0.4.3** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable tag refs rather than full 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved:
- `uses: actions/checkout@v7` (line ~57) — should be pinned to a full SHA
- `uses: actions/setup-go@v6` (line ~68) — should be pinned to a full SHA

The third reference (`dcarbone/install-jq-action@4fcb5062d7ce9bc4382d1a352d19ba3ba2c317c1`) is correctly pinned and passes.

Locations:

- `action.yml:57`
- `action.yml:68`

### github-env-injection (severity: high)

In `src/collect-info.sh`, the variable `bin_path` is constructed by concatenating the inherited process environment variable `$GITHUB_WORKSPACE` with a locally-computed suffix, and then written directly to `$GITHUB_OUTPUT` without sanitization (`printf '%s' ... | tr -d '\n\r'`). In a composite action, `$GITHUB_WORKSPACE` is set by the calling workflow and is therefore workflow-controlled/untrusted. A malicious caller could set `GITHUB_WORKSPACE` to a value containing newline characters, injecting arbitrary key=value pairs into `$GITHUB_OUTPUT` (and potentially influencing subsequent steps).

Offending line:
```sh
bin_path="$GITHUB_WORKSPACE/${bin_dir}"
echo "bin-path=${bin_path}" >> "$GITHUB_OUTPUT"   # FAIL: no tr -d newlines
```

Fix:
```sh
safe_bin_path=$(printf '%s' "$GITHUB_WORKSPACE/${bin_dir}" | tr -d '\n\r')
echo "bin-path=${safe_bin_path}" >> "$GITHUB_OUTPUT"
```

Locations:

- `src/collect-info.sh:33`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

Fixed three issues: (1) Pinned actions/checkout@v7 to commit SHA 3d3c42e5aac5ba805825da76410c181273ba90b1 in action.yml line ~57; (2) Pinned actions/setup-go@v6 to commit SHA 924ae3a1cded613372ab5595356fb5720e22ba16 in action.yml line ~68; (3) Fixed github-env-injection in src/collect-info.sh by replacing the direct echo of bin_path with a sanitized version using `printf '%s' "$GITHUB_WORKSPACE/${bin_dir}" | tr -d '\n\r'` to strip any newline characters that could be injected via the workflow-controlled GITHUB_WORKSPACE variable.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in the 'Install enry' step (action.yml line 85). The original code expanded `${DOWNLOAD_ENRY_OUTPUT_REF}` unquoted inside a double-quoted shell string, allowing command substitution if the ref value contained `$(...)`. The fix uses `printf -- '-X main.commit=%s -X main.version=%s' "${commit}" "${DOWNLOAD_ENRY_OUTPUT_REF}"` to safely embed both values as literal strings (the `%s` format specifier does not interpret shell metacharacters), stores the result in a local `ldflags` variable, and passes it to `go build` as `"${ldflags}"`. Shell variable expansion does not re-interpret command substitutions within variable contents, so the fix is safe.

