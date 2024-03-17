---
created: 2024-02-06T09:44
updated: 2024-03-11T19:49
tags:
  - Browser
  - Firefox
related link: https://mozilla.github.io/policy-templates/#passwordmanagerenabled
---
Remove access to the password manager via preferences and blocks about:logins on Firefox 70.

**Compatibility:** Firefox 70, Firefox ESR 60.2  
**CCK2 Equivalent:** N/A  
**Preferences Affected:** `pref.privacy.disable_button.view_passwords`

#### Windows (GPO)

```
Software\Policies\Mozilla\Firefox\PasswordManagerEnabled = 0x1 | 0x0
```

Addendum :

```powershell
New-Item -Path 'HKLM:\SOFTWARE\Policies\Mozilla\Firefox' -Force | New-ItemProperty -Name 'PasswordManagerEnabled' -Value 0 -PropertyType DWord -Force
```

#### Windows (Intune)

OMA-URI:

```
./Device/Vendor/MSFT/Policy/Config/Firefox~Policy~firefox/PasswordManagerEnabled
```

Value (string):

```
<enabled/> or <disabled/>
```

#### macOS

```
<dict>
  <key>PasswordManagerEnabled</key>
  <true/> | <false/>
</dict>
```

#### policies.json

```json
{
  "policies": {
    "PasswordManagerEnabled": true | false
  }
}
```