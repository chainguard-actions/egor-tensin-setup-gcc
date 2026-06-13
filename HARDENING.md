<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-gcc/v1.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **egor-tensin--setup-gcc/v1.3** was hardened automatically. 8 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

All three `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions inside PowerShell shell commands (sub-rule a). GitHub Actions substitutes these expressions into the shell command string before the shell parses it, allowing an attacker-controlled value to inject arbitrary PowerShell commands. Affected expressions across the three steps include: `${{ runner.os }}`, `${{ inputs.cygwin }}`, `${{ inputs.version }}`, `${{ inputs.platform }}` (step 1); `${{ runner.os }}`, `${{ inputs.cygwin }}`, `${{ inputs.cc }}`, `${{ steps.install.outputs.gcc }}`, `${{ steps.install.outputs.gxx }}` (step 2); `${{ inputs.cygwin }}`, `${{ inputs.hardlinks }}` (step 3). These values should be passed via `env:` variables and referenced as `$env:VAR_NAME` in PowerShell instead.

Locations:

- `action.yml:34`
- `action.yml:37`
- `action.yml:40`
- `action.yml:42`
- `action.yml:94`
- `action.yml:97`
- `action.yml:100`
- `action.yml:102`
- `action.yml:103`
- `action.yml:126`
- `action.yml:127`

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

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all script injection findings in action.yml by moving every ${{ ... }} expression out of run: blocks and into env: blocks for all three composite action steps. PowerShell code now references values via $env:VAR_NAME instead of direct expression interpolation. Step 1: RUNNER_OS, INPUT_CYGWIN, INPUT_VERSION, INPUT_PLATFORM. Step 2: RUNNER_OS, INPUT_CYGWIN, INPUT_CC, INSTALL_GCC, INSTALL_GXX. Step 3: INPUT_CYGWIN, INPUT_HARDLINKS. Output value: fields are unchanged as they are not shell commands.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in action.yml at lines 130-131. The $gcc and $gxx PowerShell variables (derived from string literals concatenated with user-controlled $version input) are now sanitized before being written to $GITHUB_OUTPUT. Added two sanitization lines using PowerShell's -replace operator to strip \r and \n characters: `$safe_gcc = $gcc -replace '[\r\n]', ''` and `$safe_gxx = $gxx -replace '[\r\n]', ''`, then writing the safe variables instead of the originals to $GITHUB_OUTPUT.

