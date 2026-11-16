<!-- markdownlint-disable -->

# Hardening Report: egor-tensin--setup-gcc/v2.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **egor-tensin--setup-gcc/v2.0.3** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the 'install' step's pwsh run: block puts a GitHub expression straight into the script: "$linux_host = '${{ runner.os }}' -eq 'Linux'" and "$windows_host = '${{ runner.os }}' -eq 'Windows'". Fix: read the value from $env:RUNNER_OS instead.

Locations:

- `action.yml:34`
- `action.yml:35`

### script-injection (severity: high)

Rule (a): the second step's pwsh run: block (the cc/gcc symlink step) puts a GitHub expression straight into the script: "$linux_host = '${{ runner.os }}' -eq 'Linux'" and "$windows_host = '${{ runner.os }}' -eq 'Windows'". Fix: read the value from $env:RUNNER_OS instead.

Locations:

- `action.yml:106`
- `action.yml:107`

### github-env-injection (severity: high)

The 'install' step builds $gcc and $gxx from the untrusted inputs.version value (via ARG_VERSION: $gcc += "-$version"). It then writes them to $GITHUB_OUTPUT with no newline stripping: echo "gcc=$gcc" >> $env:GITHUB_OUTPUT and echo "gxx=$gxx" >> $env:GITHUB_OUTPUT. A version value containing CR/LF could add extra output entries. Fix: remove \r and \n from the values (e.g. -replace '[\r\n]','') before writing them.

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

In action.yml, both pwsh steps (the install step and the cc/gcc link step) now read the OS from $env:RUNNER_OS instead of having '${{ runner.os }}' pasted into the script. In the install step, $gcc and $gxx now have \r and \n removed before they are written to $GITHUB_OUTPUT. These are the same changes the previous harden made. I haven't run the action to test these edits.

