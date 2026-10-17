<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-gcc/v2.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **egor-tensin--setup-gcc/v2.0.2** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `${{ runner.os }}` expression is interpolated directly inside the `run:` shell script string in both composite action steps. Per the security rules, ANY `${{ ... }}` expression directly inside a `run:` block is a script-injection finding — including `runner.*` contexts — because the value flows through YAML template substitution before the shell ever sees it. Step 1 contains: `$linux_host = '${{ runner.os }}' -eq 'Linux'` and `$windows_host = '${{ runner.os }}' -eq 'Windows'`. Step 2 contains the same two lines. The fix is to use the pre-set environment variable `$env:RUNNER_OS` instead of the `${{ runner.os }}` expression.

Locations:

- `action.yml:32`
- `action.yml:33`
- `action.yml:91`
- `action.yml:92`

### github-env-injection (severity: high)

In step 1, the variables `$gcc` and `$gxx` are written to `$GITHUB_OUTPUT` without sanitization. These variables are derived from `$version` (read from `$env:ARG_VERSION`, which is set to `${{ inputs.version }}` — a user-controlled input). Specifically, `$gcc += "-$version"` and `$gxx += "-$version"` incorporate the attacker-controllable `inputs.version` value. The unsanitized values are then written: `echo "gcc=$gcc" >> $env:GITHUB_OUTPUT` and `echo "gxx=$gxx" >> $env:GITHUB_OUTPUT`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write, allowing newline injection into `$GITHUB_OUTPUT`.

Locations:

- `action.yml:80`
- `action.yml:81`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed two findings in hardened/action/action.yml:
1. script-injection: Replaced `'${{ runner.os }}'` with `$env:RUNNER_OS` in both pwsh run blocks (steps 1 and 2). Since the shell is pwsh, the pre-set environment variable `$env:RUNNER_OS` is the correct replacement.
2. github-env-injection: Added PowerShell newline sanitization (`-replace '[\r\n]', ''`) for both `$gcc` and `$gxx` variables before writing them to `$GITHUB_OUTPUT`, preventing newline injection via the user-controlled `inputs.version` value.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Replaced `'${{ runner.os }}'` with `$env:RUNNER_OS` in `.github/actions/build-threads/action.yml` (line 24). Since the shell is `pwsh`, the correct syntax for accessing the runner OS environment variable is `$env:RUNNER_OS`, which is automatically provided by GitHub Actions without any template substitution. No env: block was needed since RUNNER_OS is already a standard runner environment variable.

