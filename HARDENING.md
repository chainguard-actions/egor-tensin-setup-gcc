<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-gcc/v2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **egor-tensin--setup-gcc/v2.0** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ ... }} expressions inside PowerShell shell scripts, enabling script injection. In action.yml step 1 (id: install): '${{ runner.os }}', '${{ inputs.version }}', and '${{ inputs.platform }}' are interpolated directly into the script string. In action.yml step 2: '${{ runner.os }}', '${{ inputs.cc }}', '${{ steps.install.outputs.gcc }}', and '${{ steps.install.outputs.gxx }}' are interpolated directly. In .github/actions/build-foo/action.yml: '${{ inputs.version }}', '${{ matrix.platform }}', and '${{ runner.os }}' are interpolated directly. In .github/actions/check-cc/action.yml: '${{ inputs.version }}' is interpolated directly. Any of these values can contain PowerShell metacharacters that execute arbitrary code before the shell ever sees them. Sub-rule (a): direct expression interpolation in run: blocks.

Locations:

- `action.yml:29`
- `action.yml:33`
- `action.yml:34`
- `action.yml:92`
- `action.yml:95`
- `action.yml:97`
- `action.yml:98`
- `.github/actions/build-foo/action.yml:12`
- `.github/actions/build-foo/action.yml:16`
- `.github/actions/build-foo/action.yml:21`
- `.github/actions/check-cc/action.yml:27`

### github-env-injection (severity: high)

action.yml step 1 (id: install) writes $gcc and $gxx to $env:GITHUB_OUTPUT without sanitization. These variables are derived from ${{ inputs.version }} via string concatenation (e.g. `$gcc += "-$version"` where $version comes from `${{ inputs.version }}`). An attacker-controlled input.version containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, poisoning subsequent steps. The required sanitization step (printf '%s' ... | tr -d '\n\r') is absent.

Locations:

- `action.yml:88`
- `action.yml:89`

### unpinned-uses (severity: high)

The workflow file uses actions/checkout@v6, which is a mutable tag reference rather than a pinned full-length SHA commit hash. A tag can be moved to point to a different (potentially malicious) commit, enabling supply-chain attacks. It should be pinned to a full 40-character hex SHA (e.g. actions/checkout@<sha> # v6).

Locations:

- `.github/workflows/test.yml:19`

### missing-permissions (severity: medium)

The workflow file .github/workflows/test.yml has no top-level permissions: key, and neither of its jobs (ubuntu, versions) defines a job-level permissions: block. Without explicit permissions, the workflow inherits the default repository permissions (which may include write access to contents, packages, etc.), violating the principle of least privilege.

Locations:

- `.github/workflows/test.yml:1`

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

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions, static-inline-injection

**Notes:**

Fixed all findings across 4 files:

1. action.yml (step 1 - install): Moved ${{ runner.os }}, ${{ inputs.version }}, ${{ inputs.platform }} to env: block as INPUT_OS, INPUT_VERSION, INPUT_PLATFORM. Added newline sanitization before writing to GITHUB_OUTPUT using PowerShell -replace '[\r\n]', ''.

2. action.yml (step 2): Moved ${{ runner.os }}, ${{ inputs.cc }}, ${{ steps.install.outputs.gcc }}, ${{ steps.install.outputs.gxx }} to env: block as INPUT_OS, INPUT_CC, INPUT_GCC, INPUT_GXX.

3. .github/actions/build-foo/action.yml: Moved ${{ inputs.version }}, ${{ matrix.platform }}, ${{ runner.os }} to env: block.

4. .github/actions/check-cc/action.yml: Moved ${{ inputs.version }} to env: block.

5. .github/workflows/test.yml: Pinned actions/checkout@v6 to full SHA d23441a48e516b6c34aea4fa41551a30e30af803 # v6 (both occurrences). Added top-level permissions: {} block.

