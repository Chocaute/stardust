---
created: 2023-12-06T14:20
updated: 2024-01-05T16:33
tags:
  - Windows
  - Automation
  - Deployment
  - "#Debug"
---
Hi,

when you call sysprep and it fails (logfile C:\windows\System32\sysprep\panther\setuperr.log) with an error **“Package xy installed for a user, but not provisioned for all users”** you have to remove those packages from the user profile.
  
Root cause is that the user has installed those package(s) but they are not provisioned for all users.

To solve this get a list of all provisioned appx apackages and remove the packages for all users which are not provisioned. Error can ignored..

```
$aProvPackages``=@(``Get-AppxProvisionedPackage` `-Online``).PackageName
```

```
Get-AppxPackage -AllUsers | ?{ -not ($aProvPackages -contains $_.PackageFullName ) } | %{write-host $_;Remove-AppxPackage -AllUsers -Package $_}
```

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4|`D:\>` `cd` `/d` `C:\Windows\system32\sysprep`<br><br>`C:\Windows\system32\sysprep\Actionfiles> takeown` `/F` `Generalize.xml`<br><br>`C:\Windows\system32\sysprep\Actionfiles> icacls Generalize.xml` `/grant` `myUser:(m)`<br><br>`C:\Windows\system32\sysprep\Actionfiles> notepad Generalize.xml`|

Locate the follwing XML node and remove it, save the file and start sysprep again.

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6|`<``imaging` `exclude``=``""``>`<br><br>    `<``assemblyIdentity` `name``=``"Microsoft-Windows-AppX-Sysprep"` `version``=``"10.0.19041.1566"` `publicKeyToken``=``"31bf3856ad364e35"` `processorArchitecture``=``"amd64"` `versionScope``=``"NonSxS"``/>`<br><br>    `<``sysprepOrder` `order``=``"0x1A00"``/>`<br><br>    `<``sysprepValidate` `methodName``=``"SysprepGeneralizeValidate"` `moduleName``=``"$(runtime.system32)\AppxSysprep.dll"``/>`<br><br>    `<``sysprepModule` `methodName``=``"SysprepGeneralize"` `moduleName``=``"$(runtime.system32)\AppxSysprep.dll"``/>`<br><br>`</``imaging``>`|

Michael

______________________________________

I am running into this issue as well, where the resolution steps found in this article ([https://learn.microsoft.com/en-us/troubleshoot/windows-client/deployment/sysprep-fails-remove-or-update-store-apps](https://learn.microsoft.com/en-us/troubleshoot/windows-client/deployment/sysprep-fails-remove-or-update-store-apps)) do not solve the issue.

Edit: **Solution (for now), copied from my comment below.**

#### I have been able to successfully get sysprep to work BUT it required that I never open the start menu. 
Win + X and Powershell are your friend. The issue is specifically opening the Windows Start menu because that kicks off all the Windows Store/Appx Updates on the machine. You CAN use file explorer and what not. I have a theory that you could probably also use the Cortana Search (by click the Magnifying Glass icon and NOT the start menu) as well; but haven't tested that yet.