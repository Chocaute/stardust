---
created: 2023-12-06T11:01
updated: 2024-01-05T16:33
tags:
  - Windows
  - Deployment
  - Automation
---
https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/sysprep--system-preparation--overview?view=windows-10

**Sysprep** (System Preparation) prepares a Windows client or Windows Server installation for imaging. **Sysprep** can remove PC-specific information from a Windows installation (generalizing) so it can be installed on different PCs. When you run **Sysprep** you can configure whether the PC will boot to [audit mode](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/audit-mode-overview?view=windows-11) or to the [Out-of-Box Experience (OOBE)](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/configure-oobexml?view=windows-11)
Sysprep is part of the Windows image, and is run in audit mode.

## Sysprep features

Sysprep provides the following features:

- Removes PC-specific information from the Windows image, including the PC's security identifier (SID). This allows you to capture the image and apply it to other PCs. This is known as generalizing the PC.
- Uninstalls, but doesn't remove, PC-specific drivers from the Windows image.
- Prepares the PC for delivery to a customer by setting the PC to boot to OOBE.
- Allows you to add answer file (unattend) settings to an existing installation.
## Practical uses

Runnig Sysprep helps you:

- Manage multiple PCs by creating a generic image that can be used across multiple hardware designs.
- Deploy PCs by capturing and deploying images with unique security identifiers.
- Fine-tune setup of individual PCs by adding apps, languages, or drivers in audit mode. For more information, see [Audit Mode Overview](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/audit-mode-overview?view=windows-11).
- Provide more reliable PCs by testing in audit mode before delivering them to customers.



