<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-gcc/v2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **egor-tensin--setup-gcc/v2** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): The first `run:` block (step id: install) in action.yml directly interpolates GitHub Actions expressions inside a PowerShell shell script. YAML template substitution occurs before the shell parses the script, so an attacker-controlled value can inject arbitrary PowerShell commands. Offending lines:
- Line 29: `New-Variable os -Value '${{ runner.os }}' -Option Constant`
- Line 34: `New-Variable version -Value ('${{ inputs.version }}') -Option Constant`
- Line 36: `New-Variable x64 -Value ('${{ inputs.platform }}' -eq 'x64') -Option Constant`

All `${{ ... }}` expressions must be moved to an `env:` block and referenced as environment variables inside the script.

Locations:

- `action.yml:29`
- `action.yml:34`
- `action.yml:36`

### script-injection (severity: high)

Rule (a): The second `run:` block in action.yml directly interpolates GitHub Actions expressions inside a PowerShell shell script. YAML template substitution occurs before the shell parses the script, so attacker-controlled values can inject arbitrary PowerShell commands. Offending lines:
- Line 88: `New-Variable os -Value '${{ runner.os }}' -Option Constant`
- Line 92: `New-Variable cc -Value ('${{ inputs.cc }}' -eq '1') -Option Constant`
- Line 94: `New-Variable gcc -Value '${{ steps.install.outputs.gcc }}' -Option Constant`
- Line 95: `New-Variable gxx -Value '${{ steps.install.outputs.gxx }}' -Option Constant`

All `${{ ... }}` expressions must be moved to an `env:` block and referenced as environment variables inside the script.

Locations:

- `action.yml:88`
- `action.yml:92`
- `action.yml:94`
- `action.yml:95`

### github-env-injection (severity: high)

The first `run:` block (step id: install) writes `$gcc` and `$gxx` to `$env:GITHUB_OUTPUT` without sanitization. These variables are derived from `$version`, which is directly interpolated from `${{ inputs.version }}` (an attacker-controlled input). A malicious value containing newlines could inject additional key=value pairs into GITHUB_OUTPUT. The required sanitization step (`printf '%s' ... | tr -d '\n\r'` or PowerShell equivalent) is absent before the write.
- Line 83: `echo "gcc=$gcc" >> $env:GITHUB_OUTPUT`
- Line 84: `echo "gxx=$gxx" >> $env:GITHUB_OUTPUT`

Locations:

- `action.yml:83`
- `action.yml:84`

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

Fixed all findings in action.yml:
1. Step 'install' (step 1): Moved ${{ runner.os }}, ${{ inputs.version }}, and ${{ inputs.platform }} out of the run: block into an env: block as INPUT_OS, INPUT_VERSION, INPUT_PLATFORM. PowerShell script now reads $env:INPUT_OS, $env:INPUT_VERSION, $env:INPUT_PLATFORM.
2. Step 2 (unnamed): Moved ${{ runner.os }}, ${{ inputs.cc }}, ${{ steps.install.outputs.gcc }}, ${{ steps.install.outputs.gxx }} out of the run: block into an env: block as INPUT_OS, INPUT_CC, INPUT_GCC, INPUT_GXX. PowerShell script now reads $env:INPUT_OS, $env:INPUT_CC, $env:INPUT_GCC, $env:INPUT_GXX.
3. GITHUB_OUTPUT writes: Added PowerShell newline sanitization ($safe_gcc = $gcc -replace '[\r\n]', '' and $safe_gxx = $gxx -replace '[\r\n]', '') before writing gcc and gxx values to $env:GITHUB_OUTPUT.
All ${{ }} expressions in run: blocks have been eliminated. The only remaining expressions are in value: fields (outputs section) and env: blocks, both of which are safe.

