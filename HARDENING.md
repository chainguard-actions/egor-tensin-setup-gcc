<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-gcc/v2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **egor-tensin--setup-gcc/v2.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the 'install' step's pwsh run block puts expressions straight into the script: `$linux_host = '${{ runner.os }}' -eq 'Linux'` and `$windows_host = '${{ runner.os }}' -eq 'Windows'`. Read $env:RUNNER_OS instead.

Locations:

- `action.yml:34`
- `action.yml:35`

### script-injection (severity: high)

Rule (a): the second (link) step's pwsh run block puts expressions straight into the script: `$linux_host = '${{ runner.os }}' -eq 'Linux'` and `$windows_host = '${{ runner.os }}' -eq 'Windows'`. Read $env:RUNNER_OS instead.

Locations:

- `action.yml:108`
- `action.yml:109`

### github-env-injection (severity: high)

The 'install' step builds $gcc and $gxx from the untrusted input inputs.version (ARG_VERSION), as in `$gcc += "-$version"`. It then writes them to GITHUB_OUTPUT (`echo "gcc=$gcc" >> $env:GITHUB_OUTPUT`, `echo "gxx=$gxx" >> $env:GITHUB_OUTPUT`) without first removing CR/LF characters. A version value containing a newline could inject extra output entries.

Locations:

- `action.yml:100`
- `action.yml:101`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index d60af7a..8df7fce 100644
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
@@ -94,6 +111,8 @@ runs:
             throw "Sorry, installing GCC is unsupported on $os"
         }
 
+        $gcc = $gcc -replace '[\r\n]', ''
+        $gxx = $gxx -replace '[\r\n]', ''
         echo "gcc=$gcc" >> $env:GITHUB_OUTPUT
         echo "gxx=$gxx" >> $env:GITHUB_OUTPUT
       shell: pwsh
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

I fixed all three findings in action.yml. Nothing was run to test the changes.

- **script-injection:** both pwsh steps (install and link) used to paste `${{ runner.os }}` straight into the script. They now read `$env:RUNNER_OS` instead.
- **github-env-injection:** in the install step, `$gcc` and `$gxx` now have CR/LF characters removed (`-replace '[\r\n]', ''`) before they are written to GITHUB_OUTPUT.
- **prior-fix:** the two changes above are the same ones the previous harden made, so this re-applies that patch.

