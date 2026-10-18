<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-gcc/v2.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **egor-tensin--setup-gcc/v2.0.3** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): `${{ runner.os }}` is directly interpolated inside `run:` shell command strings in both composite action steps. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it. Step 1 (lines 34–35): `$linux_host = '${{ runner.os }}' -eq 'Linux'` and `$windows_host = '${{ runner.os }}' -eq 'Windows'`. Step 2 (lines 96–97): same pattern repeated. These should be replaced with the environment variable `$env:RUNNER_OS` (already available as a pre-set GitHub Actions env var) and referenced via the env block instead of direct expression interpolation.

Locations:

- `action.yml:34`
- `action.yml:35`
- `action.yml:96`
- `action.yml:97`

### github-env-injection (severity: high)

The first `run:` block writes `$gcc` and `$gxx` to `$GITHUB_OUTPUT` without sanitization. These variables are derived from `$version`, which is read from `ARG_VERSION` (set to `${{ inputs.version }}`—an untrusted caller-controlled input). The script builds `$gcc += "-$version"` and `$gxx += "-$version"` and then writes `echo "gcc=$gcc" >> $env:GITHUB_OUTPUT` and `echo "gxx=$gxx" >> $env:GITHUB_OUTPUT`. No newline-stripping sanitization (equivalent of `printf '%s' ... | tr -d '\n\r'`) is applied before the write, allowing a crafted `inputs.version` value containing newlines to inject arbitrary key=value pairs into GITHUB_OUTPUT.

Locations:

- `action.yml:89`
- `action.yml:90`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed two security findings in hardened/action/action.yml:
1. script-injection: Replaced '${{ runner.os }}' direct interpolation in both run: blocks (steps 1 and 2) with $env:RUNNER_OS, the pre-set GitHub Actions environment variable, using proper PowerShell syntax.
2. github-env-injection: Added newline sanitization before writing gcc and gxx values to GITHUB_OUTPUT using PowerShell's -replace '[\r\n]', '' to prevent injection via a crafted inputs.version value.

