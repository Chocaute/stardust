---
created: 2024-02-06T10:01
updated: 2024-03-11T19:47
tags:
  - Browser
  - Chrome
related link: https://blog.httpwatch.com/2021/08/06/how-to-automatically-install-and-enable-a-chrome-extension/
---
Il faut ajouter une clef dans le registre windows :

```r
[HKEY_LOCAL_MACHINE\Software\Policies\Google\Chrome\ExtensionInstallForcelist] = "1"="nngceckbapebfimnlniiiahkandclblb"
```