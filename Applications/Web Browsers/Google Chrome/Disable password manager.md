---
created: 2024-02-06T09:58
updated: 2024-03-11T19:46
tags:
  - Browser
  - Chrome
related link: https://www.tenforums.com/tutorials/115669-enable-disable-saving-passwords-google-chrome-windows.html
---
Il faut ajouter une clef dans le registre windows :

```r
[HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Google\Chrome] = "PasswordManagerEnabled"=dword:00000000
```


