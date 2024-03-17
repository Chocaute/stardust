---
created: 2024-02-22T14:52
updated: 2024-02-22T14:55
tags:
  - WebHosting
  - GLPI
  - Windows
related link: https://glpi-agent.readthedocs.io/en/latest/installation/windows-command-line.html
---
By default, the installer will bring you to the graphical user interface unless you use the /i /quiet options, calling it from command line.

```cmd
GLPI-Agent-1.7-x64.msi /quiet SERVER=<URL>
```

```cmd
msiexec /i GLPI-Agent-1.7-x64.msi /quiet SERVER=<URL>
```

The latter seems to work more often than the first one.

> [!NOTE]
> Don't use **PowerShell** as commandline interpreter: it changes some environment context the installer doesn't handle well.

