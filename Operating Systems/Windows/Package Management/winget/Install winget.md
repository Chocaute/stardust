---
created: 2024-02-22T14:40
updated: 2024-02-22T14:41
tags:
  - Windows
  - winget
---
https://github.com/microsoft/winget-cli/issues/3832#issuecomment-1872387214

```
Invoke-WebRequest -Uri https://aka.ms/getwinget -OutFile winget.msixbundle
Add-AppPackage -ForceApplicationShutdown .\winget.msixbundle
del .\winget.msixbundle
```


