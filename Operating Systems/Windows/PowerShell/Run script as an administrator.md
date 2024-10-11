---
created: 2024-02-22T17:05
updated: 2024-02-22T17:11
tags:
  - Powershell
  - Windows
related link: https://superuser.com/a/532109
---
On UAC-enabled systems, to make sure a script is running with full admin privileges, add this code at the beginning of your script:

```powershell
param([switch]$Elevated)

function Test-Admin {
    $currentUser = New-Object Security.Principal.WindowsPrincipal $([Security.Principal.WindowsIdentity]::GetCurrent())
    $currentUser.IsInRole([Security.Principal.WindowsBuiltinRole]::Administrator)
}

if ((Test-Admin) -eq $false)  {
    if ($elevated) {
        # tried to elevate, did not work, aborting
    } else {
        Start-Process powershell.exe -Verb RunAs -ArgumentList ('-noprofile -noexit -file "{0}" -elevated' -f ($myinvocation.MyCommand.Definition))
    }
    exit
}

'running with full privileges'
```

Now, when running your script, it will call itself again and attempt to elevate privileges before running. The -elevated switch prevents it from repeating if something fails.

You may remove the `-noexit` switch if the terminal should automatically close when the script finishes.