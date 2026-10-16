<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-gcc/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **egor-tensin--setup-gcc/v2.0.1** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Step 1 (id: install) directly interpolates ${{ }} expressions inside a PowerShell run: block. The expressions ${{ inputs.version }} (line 31) and ${{ inputs.platform }} (line 32) are attacker-controlled inputs that are substituted by the GitHub Actions template engine before PowerShell parses the script, enabling arbitrary PowerShell injection. Additionally, ${{ runner.os }} (line 27) is also interpolated directly. All ${{ ... }} expressions inside run: blocks are script-injection violations regardless of source. Safe fix: pass values via environment variables (env: block) and reference them as $env:VAR_NAME in the script.

Locations:

- `action.yml:27`
- `action.yml:31`
- `action.yml:32`

### script-injection (severity: high)

Step 2 (unnamed) directly interpolates ${{ }} expressions inside a PowerShell run: block. The expressions ${{ inputs.cc }} (line 94), ${{ steps.install.outputs.gcc }} (line 96), and ${{ steps.install.outputs.gxx }} (line 97) are substituted by the template engine before PowerShell parses the script, enabling arbitrary PowerShell injection. Additionally, ${{ runner.os }} (line 90) is also interpolated directly. Safe fix: pass values via environment variables (env: block) and reference them as $env:VAR_NAME in the script.

Locations:

- `action.yml:90`
- `action.yml:94`
- `action.yml:96`
- `action.yml:97`

### github-env-injection (severity: high)

Step 1 (id: install) writes $gcc and $gxx to $env:GITHUB_OUTPUT (lines 86–87) without sanitization. These variables are derived from the attacker-controlled input ${{ inputs.version }}: when a non-latest version is requested, the script appends $version (which holds the raw inputs.version value) to the package/binary names via string concatenation (e.g. $gcc += "-$version"). A newline embedded in inputs.version could inject additional key=value pairs into GITHUB_OUTPUT. The required sanitization step (equivalent of printf '%s' ... | tr -d '\n\r') is absent before the write.

Locations:

- `action.yml:86`
- `action.yml:87`

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

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all script-injection findings in both PowerShell steps by moving ${{ }} expressions to env: blocks and referencing them as $env:VAR_NAME in the scripts. Step 1: moved runner.os, inputs.version, inputs.platform to env: (RUNNER_OS_VAL, INPUT_VERSION, INPUT_PLATFORM). Step 2: moved runner.os, inputs.cc, steps.install.outputs.gcc, steps.install.outputs.gxx to env: (RUNNER_OS_VAL, INPUT_CC, INSTALL_GCC, INSTALL_GXX). Fixed github-env-injection by sanitizing $gcc and $gxx with PowerShell's -replace '[\r\n]', '' before writing to $env:GITHUB_OUTPUT.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in two composite action files:

1. `.github/actions/build-threads/action.yml`: Moved three expressions to `env:` block — `${{ inputs.version }}` → `INPUT_VERSION`, `${{ matrix.platform }}` → `MATRIX_PLATFORM`, `${{ runner.os }}` → `RUNNER_OS_VAR`. Updated PowerShell script to reference them as `$env:INPUT_VERSION`, `$env:MATRIX_PLATFORM`, and `$env:RUNNER_OS_VAR` respectively.

2. `.github/actions/check-cc/action.yml`: Moved `${{ inputs.version }}` to `env:` block as `INPUT_VERSION`. Updated PowerShell script to reference it as `$env:INPUT_VERSION` instead of the inline expression.

Used `$env:VAR_NAME` syntax (not `$VAR_NAME`) since both steps use `shell: pwsh` (PowerShell).

