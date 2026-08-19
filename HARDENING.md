<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-gcc/v2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **egor-tensin--setup-gcc/v2** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: Multiple ${{ }} expressions are directly interpolated inside run: shell command strings. In action.yml step 1 (id: install), the PowerShell script embeds '${{ runner.os }}', '${{ inputs.version }}', and '${{ inputs.platform }}' directly in the run: block. In action.yml step 2, '${{ runner.os }}', '${{ inputs.cc }}', '${{ steps.install.outputs.gcc }}', and '${{ steps.install.outputs.gxx }}' are directly interpolated. These values flow through YAML template substitution before the shell processes them, enabling script injection. The same issue exists in .github/actions/build-foo/action.yml ('${{ inputs.version }}', '${{ matrix.platform }}', '${{ runner.os }}') and .github/actions/check-cc/action.yml ('${{ inputs.version }}').

Locations:

- `action.yml:27`
- `action.yml:31`
- `action.yml:33`
- `action.yml:75`
- `action.yml:77`
- `action.yml:78`
- `.github/actions/build-foo/action.yml:12`
- `.github/actions/build-foo/action.yml:16`
- `.github/actions/build-foo/action.yml:20`
- `.github/actions/check-cc/action.yml:27`

### github-env-injection (severity: high)

In action.yml's first step (id: install), the variables $gcc and $gxx are written to $GITHUB_OUTPUT without sanitization. These variables are derived from $version, which is set from '${{ inputs.version }}' — an untrusted user-controlled input. An attacker could inject newlines into the version input to poison GITHUB_OUTPUT with arbitrary key-value pairs. The required sanitization step (printf '%s' "$VAR" | tr -d '\n\r') is absent before the writes: `echo "gcc=$gcc" >> $env:GITHUB_OUTPUT` and `echo "gxx=$gxx" >> $env:GITHUB_OUTPUT`.

Locations:

- `action.yml:64`
- `action.yml:65`

### missing-permissions (severity: medium)

The workflow file .github/workflows/test.yml has no top-level 'permissions:' key, and neither of its jobs ('ubuntu', 'versions') defines a job-level 'permissions:' block. Without explicit permissions, the workflow runs with the default repository token permissions, which may be overly broad (e.g., write access to contents). Minimal explicit permissions should be declared.

Locations:

- `.github/workflows/test.yml:1`

### unpinned-uses (severity: high)

The workflow file .github/workflows/test.yml references 'actions/checkout@v6' using a mutable tag rather than a pinned 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit, enabling supply-chain attacks. It should be pinned to a full SHA, e.g. 'actions/checkout@<40-char-sha> # v6'.

Locations:

- `.github/workflows/test.yml:20`
- `.github/workflows/test.yml:42`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:36`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.platform }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:38`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cc }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:106`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, missing-permissions, unpinned-uses, static-inline-injection

**Notes:**

Fixed all findings across 4 files:

1. action.yml (steps 1 & 2): Moved all ${{ }} expressions (${{ runner.os }}, ${{ inputs.version }}, ${{ inputs.platform }}, ${{ inputs.cc }}, ${{ steps.install.outputs.gcc }}, ${{ steps.install.outputs.gxx }}) out of run: blocks into env: blocks. Scripts now use $env:INPUT_* variables. Also sanitized GITHUB_OUTPUT writes using PowerShell's -replace '[\r\n]', '' to strip newlines from gcc/gxx values before writing.

2. .github/actions/build-foo/action.yml: Moved ${{ inputs.version }}, ${{ matrix.platform }}, ${{ runner.os }} to env: block.

3. .github/actions/check-cc/action.yml: Moved ${{ inputs.version }} to env: block.

4. .github/workflows/test.yml: Added top-level `permissions: {}` and pinned both actions/checkout@v6 references to full SHA d23441a48e516b6c34aea4fa41551a30e30af803 # v6.

