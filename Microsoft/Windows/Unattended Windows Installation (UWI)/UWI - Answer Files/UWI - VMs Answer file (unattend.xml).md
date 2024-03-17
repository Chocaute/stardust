---
created: 2023-12-06T10:52
updated: 2024-01-05T16:32
tags:
  - Windows
  - Automation
  - Deployment
---
https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/windows-setup-automation-overview?view=windows-10#implicit-answer-file-search-order

Answer files (or Unattend files) can be used to modify Windows settings in your images during Setup. You can also create settings that trigger scripts in your images that run after the first user creates their account and picks their default language.

_______

On how to create a catalog file -> https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/update-windows-settings-and-scripts-create-your-own-answer-file-sxs?view=windows-10#step-1-create-a-catalog-file

You need : 
- Windows System Image Manager, avaible in Windows ADK. 
- A valid install.wim inside a windows image.
  
  _____________

Windows Setup will automatically search for [answer files in certain locations](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/windows-setup-automation-overview?view=windows-11#implicit-answer-file-search-order), or you can specify an unattend file to use by using the `/unattend:` option when [running Windows Setup (setup.exe)](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/windows-setup-command-line-options?view=windows-11#unattend).

|Search Order|Location|Description|
|:--|:--|:--|
|1|Registry<br><br>HKEY_LOCAL_MACHINE\System\Setup\UnattendFile|Specifies a pointer in the registry to an answer file. The answer file is not required to be named Unattend.xml.|
|2|%WINDIR%\Panther\Unattend|The name of the answer file must be either Unattend.xml or Autounattend.xml.<br><br>**Note**  <br><br>Windows Setup searches this directory only on downlevel installations. If Windows Setup starts from Windows PE, the %WINDIR%\Panther\Unattend directory is not searched.|

## Sensitive Data in Answer Files

Setup removes sensitive data in the cached answer file at the end of each configuration pass.

Because answer files are cached to the computer during Windows Setup, your answer files will persist on the computer between reboots. Before you deliver the computer to a customer, you must delete the cached answer file in the %WINDIR%\panther directory. There might be potential security issues if you include domain passwords, product keys, or other sensitive data in your answer file.

## Improve Security for Answer Files

Answer files store sensitive data, including product keys, passwords, and other account information. You can help protect this sensitive data by following these best practices:

- **Restrict access to answer files.** Depending on your environment, you can change the access control lists (**ACLs**) or permissions on a file. Only approved accounts can access answer files.
    
- **Hide passwords.** To improve security in answer files, you can hide the passwords for local accounts by using Windows SIM. For more information, see [Hide Sensitive Data in an Answer File](https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/wsim/hide-sensitive-data-in-an-answer-file).
    
- **Delete the cached answer file.** During unattended Windows installation, answer files are cached to the computer. For each configuration pass, sensitive information such as domain passwords and product keys are deleted in the cached answer file. However, other information is still readable in the answer file. Before you deliver the computer to a customer, delete the cached answer file in **%WINDIR%\panther**.

# Hide Sensitive Data in an Answer File

You can use Windows® System Image Manager (Windows SIM) to hide the password for the administrator account, and for any other user accounts on the local system, in an answer file. Hiding passwords in an answer file prevents users from reading the answer file and identifying passwords for local accounts.

The settings that you can hide include the following:

- **Microsoft-Windows-Shell-Setup | AutoLogon | Password**
- **Microsoft-Windows-Shell-Setup | UserAccounts | AdministratorPassword**
- **Microsoft-Windows-Shell-Setup | UserAccounts | LocalAccounts | LocalAccount | Password**

On the **Tools** menu, click **Hide Sensitive Data**. This makes sure that when the answer file is saved, the password information will be hidden.

______________________________________

# Automate OOBE

You can use Unattend settings to prevent some or all of the user interface (UI) pages from appearing in Windows OOBE. To fully automate OOBE, use Unattend to configure what a user would normally configure during OOBE. OOBE screens that aren't configured in Unattend will display when a user goes through OOBE.

To learn more about creating an answer file using Unattend, as well as a full list of Unattend settings available to you, see the [Unattended Windows Setup Reference](https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/).

Customize the following Unattend settings to automate OOBE:

|Setting|Configuration pass|Description|
|---|---|---|
|Microsoft-Windows-International-Core settings:<br><br>- [InputLocale](https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/microsoft-windows-international-core-inputlocale)<br>- [SystemLocale](https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/microsoft-windows-international-core-systemlocale)<br>- [UILanguage](https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/microsoft-windows-international-core-uilanguage)<br>- [UserLocale](https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/microsoft-windows-international-core-userlocale)|oobeSystem|Specifies the region-specific defaults of the Windows installation.|
|[Microsoft-Windows-Shell-Setup/UserAccounts](https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/microsoft-windows-shell-setup-useraccounts)|oobeSystem|Specifies the user accounts, and passwords, to create on the Windows installation. The account can be a user account, a domain account, or the default administrator account.|
|Microsoft-Windows-Shell-Setup/OOBE settings:<br><br>- [HideEULAPage](https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/microsoft-windows-shell-setup-oobe-hideeulapage)<br>- [HideOEMRegistrationScreen](https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/microsoft-windows-shell-setup-oobe-hideoemregistrationscreen)<br>- [HideOnlineAccountScreens](https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/microsoft-windows-shell-setup-oobe-hideonlineaccountscreens)<br>- [HideWirelessSetupInOOBE](https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/microsoft-windows-shell-setup-oobe-hidewirelesssetupinoobe)<br>- [HideLocalAccountScreen](https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/microsoft-windows-shell-setup-oobe-hidelocalaccountscreen)|oobeSystem|Specifies whether certain OOBE screens will be hidden.|
|[Microsoft-Windows-Shell-Setup/OOBE/ProtectYourPC](https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/microsoft-windows-shell-setup-oobe-protectyourpc)|oobeSystem|Specifies whether your device is configured with Express settings, such as sending data to Microsoft, letting Windows and apps request the user's localization, and turning on protection against malicious web content.|