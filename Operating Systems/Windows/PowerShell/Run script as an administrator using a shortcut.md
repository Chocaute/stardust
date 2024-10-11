---
created: 2024-02-22T17:04
updated: 2024-02-22T17:05
tags:
  - Powershell
  - Windows
related link: https://stackoverflow.com/a/66364332
---
# Tl;dr

This will do the trick:

```powershell
powershell.exe -Command "& {$wd = Get-Location; Start-Process powershell.exe -Verb RunAs -ArgumentList \"-ExecutionPolicy ByPass -NoExit -Command Set-Location $wd; C:\project\test.ps1\"}"
```

---

# Explanation

First, you have to call PowerShell to be able to execute `Start-Process`. You don't need any additional paramters at this point, because you just use this first PowerShell to launch another one. You do it like this:

```bash
powershell.exe -Command "& {...}"
```

Inside the curly braces you can insert any script block. First you will retrieve your current working directory (CWD) to set it in the new launched PowerShell. Then you call PowerShell with `Start-Process` and add the `-Verb RunAs` parameter to elevate it:

```sql
$wd = Get-Location; Start-Process powershell.exe -Verb RunAs -ArgumentList ...
```

Then you need to add all desired PowerShell parameters to the `ArgumentList`. In your case, these will be:

```erlang
-ExecutionPolicy ByPass -NoExit -Command ...
```

Finally, you pass the commands that you want to execute to the `-Command` parameter. Basically, you want to call your script file. But before doing so, you will set your CWD to the previously retrieved directory and THEN call your script:

```bash
Set-Location $wd; C:\project\test.ps1
```

In total:

```swift
powershell.exe -Command "& {$wd = Get-Location; Start-Process powershell.exe -Verb RunAs -ArgumentList \"-ExecutionPolicy ByPass -NoExit -Command Set-Location $wd; C:\project\test.ps1\"}"
```