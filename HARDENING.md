<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-gcc/v2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **egor-tensin--setup-gcc/v2** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the expression ${{ runner.os }} is pasted straight into the pwsh run: script of the 'install' step: `$linux_host = '${{ runner.os }}' -eq 'Linux'` and `$windows_host = '${{ runner.os }}' -eq 'Windows'`. Use $env:RUNNER_OS instead.

Locations:

- `action.yml:34`
- `action.yml:35`

### script-injection (severity: high)

Rule (a): the expression ${{ runner.os }} is pasted straight into the pwsh run: script of the second (link) step: `$linux_host = '${{ runner.os }}' -eq 'Linux'` and `$windows_host = '${{ runner.os }}' -eq 'Windows'`. Use $env:RUNNER_OS instead.

Locations:

- `action.yml:106`
- `action.yml:107`

### github-env-injection (severity: high)

The 'install' step builds $gcc and $gxx from the untrusted inputs.version (via env ARG_VERSION, with `$gcc += "-$version"`). It then writes them to $GITHUB_OUTPUT (`echo "gcc=$gcc" >> $env:GITHUB_OUTPUT`, `echo "gxx=$gxx" >> $env:GITHUB_OUTPUT`) without removing newlines first. A version value that contains newlines could inject extra output entries.

Locations:

- `action.yml:97`
- `action.yml:98`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index d60af7a..778e4e5 100644
--- a/action.yml
+++ b/action.yml
@@ -26,13 +26,30 @@ outputs:
 runs:
   using: composite
   steps:
     - id: install
       env:
         ARG_VERSION: ${{ inputs.version }}
         ARG_PLATFORM: ${{ inputs.platform }}
       run: |
-        $linux_host = '${{ runner.os }}' -eq 'Linux'
-        $windows_host = '${{ runner.os }}' -eq 'Windows'
+        $linux_host = $env:RUNNER_OS -eq 'Linux'
+        $windows_host = $env:RUNNER_OS -eq 'Windows'
 
         $version = (Get-ChildItem env:ARG_VERSION).Value
         $latest = $version -eq 'latest'
@@ -94,8 +111,10 @@ runs:
             throw "Sorry, installing GCC is unsupported on $os"
         }
 
-        echo "gcc=$gcc" >> $env:GITHUB_OUTPUT
-        echo "gxx=$gxx" >> $env:GITHUB_OUTPUT
+        $safe_gcc = $gcc -replace '[\r\n]', ''
+        $safe_gxx = $gxx -replace '[\r\n]', ''
+        echo "gcc=$safe_gcc" >> $env:GITHUB_OUTPUT
+        echo "gxx=$safe_gxx" >> $env:GITHUB_OUTPUT
       shell: pwsh
 
     - env:
@@ -103,8 +122,8 @@ runs:
         GCC: ${{ steps.install.outputs.gcc }}
         GXX: ${{ steps.install.outputs.gxx }}
       run: |
-        $linux_host = '${{ runner.os }}' -eq 'Linux'
-        $windows_host = '${{ runner.os }}' -eq 'Windows'
+        $linux_host = $env:RUNNER_OS -eq 'Linux'
+        $windows_host = $env:RUNNER_OS -eq 'Windows'
 
         $cc = (Get-ChildItem env:ARG_CC).Value -eq '1'
 

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, prior-fix

**Notes:**

I fixed all three findings in action.yml and haven't run the action since.

- **script-injection:** Both pwsh steps pasted `${{ runner.os }}` straight into the script. They now read `$env:RUNNER_OS`, so no expression is pasted into the script text. The step has to keep `shell: pwsh`, because `$env:` is PowerShell syntax.
- **github-env-injection:** The install step now strips newline characters from `$gcc` and `$gxx` (`-replace '[\r\n]', ''`) before writing them to `$GITHUB_OUTPUT`.
- **prior-fix:** These are the same changes the previous harden made, applied again in the same form.

