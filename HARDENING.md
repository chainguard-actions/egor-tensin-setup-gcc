<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-gcc/v2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **egor-tensin--setup-gcc/v2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): both run steps in this composite action put a GitHub Actions expression straight into the script: `$linux_host = '${{ runner.os }}' -eq 'Linux'` and `$windows_host = '${{ runner.os }}' -eq 'Windows'`. The value is pasted into the script before PowerShell runs. Use $env:RUNNER_OS instead.

Locations:

- `action.yml:34`
- `action.yml:35`
- `action.yml:106`
- `action.yml:107`

### github-env-injection (severity: high)

The 'install' step builds $gcc and $gxx from the version input (`$gcc += "-$version"`, where $version comes from env ARG_VERSION = ${{ inputs.version }}). It then writes them with `echo "gcc=$gcc" >> $env:GITHUB_OUTPUT` and `echo "gxx=$gxx" >> $env:GITHUB_OUTPUT` without removing newline or carriage-return characters first. A version value containing a newline could add extra outputs of the attacker's choosing.

Locations:

- `action.yml:97`
- `action.yml:98`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Replaced '${{ runner.os }}' with $env:RUNNER_OS in both pwsh steps. Stripped CR/LF from $gcc/$gxx (via -replace '[\r\n]','') before writing to GITHUB_OUTPUT.

