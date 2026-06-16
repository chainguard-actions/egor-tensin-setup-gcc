<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-gcc/v2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **egor-tensin--setup-gcc/v2.0** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ }} expressions are interpolated directly inside run: shell command strings in action.yml. This allows an attacker-controlled value to be injected into the PowerShell script before the shell ever sees it. Offending lines in the first run block (step id: install): `New-Variable os -Value '${{ runner.os }}'` (line 31), `New-Variable version -Value ('${{ inputs.version }}')` (line 35), `New-Variable x64 -Value ('${{ inputs.platform }}' -eq 'x64')` (line 37). Offending lines in the second run block: `New-Variable os -Value '${{ runner.os }}'` (line 93), `New-Variable cc -Value ('${{ inputs.cc }}' -eq '1')` (line 97), `New-Variable gcc -Value '${{ steps.install.outputs.gcc }}'` (line 99), `New-Variable gxx -Value '${{ steps.install.outputs.gxx }}'` (line 100). All of these should be passed via env: variables and referenced as PowerShell environment variables instead.

Locations:

- `action.yml:31`
- `action.yml:35`
- `action.yml:37`
- `action.yml:93`
- `action.yml:97`
- `action.yml:99`
- `action.yml:100`

### github-env-injection (severity: high)

The first run: block (step id: install) writes `$gcc` and `$gxx` to $GITHUB_OUTPUT without sanitization. These variables are derived from `${{ inputs.version }}` (via `$version`) and `${{ inputs.platform }}` (via `$x64`), which are attacker-controlled inputs. A malicious value containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT. The required sanitization step (`printf '%s' ... | tr -d '\n\r'`) is absent before the writes on lines 88–89. Note: this is PowerShell, so the equivalent safe pattern would be stripping newlines before writing.

Locations:

- `action.yml:88`
- `action.yml:89`

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

Rewrote action.yml to fix all findings:
1. script-injection: Moved all ${{ }} expressions out of both run: blocks into env: blocks. Step 1 (install): INPUT_OS=${{ runner.os }}, INPUT_VERSION=${{ inputs.version }}, INPUT_PLATFORM=${{ inputs.platform }}. Step 2: INPUT_OS=${{ runner.os }}, INPUT_CC=${{ inputs.cc }}, INPUT_GCC=${{ steps.install.outputs.gcc }}, INPUT_GXX=${{ steps.install.outputs.gxx }}. All referenced as $env:VAR_NAME in PowerShell.
2. github-env-injection: Added PowerShell newline sanitization ($safe_gcc = $gcc -replace '[\r\n]', '' and $safe_gxx = $gxx -replace '[\r\n]', '') before writing to $GITHUB_OUTPUT.
3. static-inline-injection: Covered by the same env: block fixes as script-injection.

