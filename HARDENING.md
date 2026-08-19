<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-gcc/v1.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **egor-tensin--setup-gcc/v1.3** was hardened automatically. 10 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Three run: blocks in action.yml directly interpolate GitHub Actions expressions (${{ ... }}) inside PowerShell shell commands, violating sub-rule (a). Step 1 (id: install) uses: `${{ runner.os }}`, `${{ inputs.cygwin }}`, `${{ inputs.version }}`, `${{ inputs.platform }}`. Step 2 uses: `${{ runner.os }}`, `${{ inputs.cygwin }}`, `${{ inputs.cc }}`, `${{ steps.install.outputs.gcc }}`, `${{ steps.install.outputs.gxx }}`. Step 3 uses: `${{ inputs.cygwin }}`, `${{ inputs.hardlinks }}`. Any ${{ ... }} expression interpolated directly into a run: block allows an attacker-controlled value to be parsed as shell/PowerShell syntax before quoting can protect it.

Locations:

- `action.yml:37`
- `action.yml:40`
- `action.yml:43`
- `action.yml:45`
- `action.yml:101`
- `action.yml:104`
- `action.yml:107`
- `action.yml:109`
- `action.yml:110`
- `action.yml:136`
- `action.yml:137`

### unpinned-uses (severity: high)

The workflow file references three external actions using mutable tag refs instead of full 40-character SHA digests, making the workflow vulnerable to supply-chain attacks if those tags are moved: `actions/checkout@v3`, `egor-tensin/cleanup-path@v3`, `egor-tensin/setup-cygwin@v4`.

Locations:

- `.github/workflows/test.yml:18`
- `.github/workflows/test.yml:50`
- `.github/workflows/test.yml:72`
- `.github/workflows/test.yml:74`
- `.github/workflows/test.yml:76`

### missing-permissions (severity: medium)

The workflow file .github/workflows/test.yml has no top-level `permissions:` key and none of its three jobs (ubuntu, versions, cygwin) define a job-level `permissions:` block. Without explicit permissions, the workflow inherits the default repository token permissions, which may be overly broad (write access to contents, packages, etc.).

Locations:

- `.github/workflows/test.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cygwin }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:42`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:45`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.platform }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:47`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cygwin }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:131`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cc }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:134`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.cygwin }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:174`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.hardlinks }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:175`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all findings in action.yml and .github/workflows/test.yml:

1. script-injection/static-inline-injection (action.yml): Moved all ${{ }} expressions out of run: blocks into env: blocks for all three composite action steps. PowerShell scripts now reference values via $env:VAR_NAME instead of direct interpolation.

2. unpinned-uses (test.yml): Pinned actions/checkout@v3, egor-tensin/cleanup-path@v3, and egor-tensin/setup-cygwin@v4 to their full 40-character commit SHAs with tag comments for readability.

3. missing-permissions (test.yml): Added top-level `permissions: {}` to restrict the workflow to no permissions by default.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection findings by moving ${{ }} expressions out of run: shell strings into env: blocks and referencing them as environment variables. Specifically: (1) build-foo/action.yml: moved inputs.version, matrix.platform, runner.os to env vars INPUT_VERSION, MATRIX_PLATFORM, RUNNER_OS; (2) build-foo-cygwin/action.yml: moved github.workspace and matrix.platform to env vars GITHUB_WORKSPACE_PATH and MATRIX_PLATFORM, replacing the heredoc workaround with direct env var reference; (3) check-cc/action.yml: moved inputs.version to env var INPUT_VERSION. Fixed github-env-injection in action.yml by sanitizing $gcc and $gxx with PowerShell's -replace '[\r\n]','' before writing to $GITHUB_OUTPUT.

