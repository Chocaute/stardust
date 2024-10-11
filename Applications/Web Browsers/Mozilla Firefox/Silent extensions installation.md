---
created: 2024-02-06T09:55
updated: 2024-03-11T19:50
tags:
  - Browser
  - Firefox
related link: https://mozilla.github.io/policy-templates/#extensions
---
Control the installation, uninstallation and locking of extensions.

While this policy is not technically deprecated, it is recommended that you use the **[`ExtensionSettings`](https://mozilla.github.io/policy-templates/#extensionsettings)** policy. It has the same functionality and adds more. It does not support native paths, though, so you’ll have to use file:/// URLs.

`Install` is a list of URLs or native paths for extensions to be installed.

`Uninstall` is a list of extension IDs that should be uninstalled if found.

`Locked` is a list of extension IDs that the user cannot disable or uninstall.

**Compatibility:** Firefox 60, Firefox ESR 60  
**CCK2 Equivalent:** `addons`  
**Preferences Affected:** N/A

#### Windows (GPO)
```
Software\Policies\Mozilla\Firefox\Extensions\Install\1 = "https://addons.mozilla.org/firefox/downloads/somefile.xpi"
Software\Policies\Mozilla\Firefox\Extensions\Install\2 = "//path/to/xpi"
Software\Policies\Mozilla\Firefox\Extensions\Uninstall\1 = "bad_addon_id@mozilla.org"
Software\Policies\Mozilla\Firefox\Extensions\Locked\1 = "addon_id@mozilla.org"
```

#### Windows (Intune)

OMA-URI:

```
./Device/Vendor/MSFT/Policy/Config/Firefox~Policy~firefox~Extensions/Extensions_Install
```

Value (string):

```
<enabled/>
<data id="Extensions" value="1&#xF000;https://addons.mozilla.org/firefox/downloads/somefile.xpi&#xF000;2&#xF000;//path/to/xpi"/>
```

OMA-URI:

```
./Device/Vendor/MSFT/Policy/Config/Firefox~Policy~firefox~Extensions/Extensions_Uninstall
```

Value (string):

```
<enabled/>
<data id="Extensions" value="1&#xF000;bad_addon_id@mozilla.org"/>
```

OMA-URI:

```
./Device/Vendor/MSFT/Policy/Config/Firefox~Policy~firefox~Extensions/Extensions_Locked
```

Value (string):

```
<enabled/>
<data id="Extensions" value="1&#xF000;addon_id@mozilla.org"/>
```

#### macOS

```
<dict>
  <key>Extensions</key>
  <dict>
    <key>Install</key>
    <array>
      <string>https://addons.mozilla.org/firefox/downloads/somefile.xpi</string>
      <string>//path/to/xpi</string>
    </array>
    <key>Uninstall</key>
    <array>
      <string>bad_addon_id@mozilla.org</string>
    </array>
    <key>Locked</key>
    <array>
      <string>addon_id@mozilla.org</string>
    </array>
  </dict>
</dict>
```

#### policies.json

```json
{
  "policies": {
    "Extensions": {
      "Install": ["https://addons.mozilla.org/firefox/downloads/somefile.xpi", "//path/to/xpi"],
      "Uninstall": ["bad_addon_id@mozilla.org"],
      "Locked":  ["addon_id@mozilla.org"]
    }
  }
}
```