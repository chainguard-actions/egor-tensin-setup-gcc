<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-gcc/v2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **egor-tensin--setup-gcc/v2.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): a GitHub Actions expression is placed directly in a run: script. The 'install' step has `$linux_host = '${{ runner.os }}' -eq 'Linux'` and `$windows_host = '${{ runner.os }}' -eq 'Windows'`. Fix: use the $env:RUNNER_OS environment variable instead.

Locations:

- `action.yml:34`
- `action.yml:35`

### script-injection (severity: high)

Rule (a): a GitHub Actions expression is placed directly in a run: script. The link step (the second step) has `$linux_host = '${{ runner.os }}' -eq 'Linux'` and `$windows_host = '${{ runner.os }}' -eq 'Windows'`. Fix: use the $env:RUNNER_OS environment variable instead.

Locations:

- `action.yml:106`
- `action.yml:107`

### github-env-injection (severity: high)

The 'install' step builds $gcc and $gxx from the untrusted inputs.version value (passed in through ARG_VERSION as `$gcc += "-$version"`). It then writes them with `echo "gcc=$gcc" >> $env:GITHUB_OUTPUT` and `echo "gxx=$gxx" >> $env:GITHUB_OUTPUT` without removing CR/LF first. A version string containing a newline could inject extra step outputs. Fix: strip \n and \r from the value before writing it.

Locations:

- `action.yml:97`
- `action.yml:98`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

In both pwsh steps, '${{ runner.os }}' was replaced with $env:RUNNER_OS. In the install step, $gcc and $gxx now have CR/LF stripped (-replace '[\r\n]', '') before they are written to GITHUB_OUTPUT.

