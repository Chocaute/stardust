---
created: 2024-02-23T09:28
updated: 2024-02-23T09:36
tags:
  - Windows
  - AD
  - GPO
  - Chrome
related link: https://www.anoopcnair.com/configure-chrome-group-policy-admx-templates/
---
Download the latest build and version of ADMX Templates for Google Chrome from [https://chromeenterprise.google/browser/download/#manage-policies-tab](https://chromeenterprise.google/browser/download/#manage-policies-tab)

Open the extracted folder **“Policy_Templates”**, and you will see the following files available:
- **Chromeos –** ADMX templates for Chromium
- **common** – It contains the **chrome_policy_atomic_groups_list.html** and **chrome_policy_list.html** two files listed with all policies.
- **windows** – Contains Chrome **ADM**/**ADMX** policy templates. 
- **VERSION** – Details of downloaded Policy for Google chrome version.

![[Pasted image 20240223092959.png]]

> [!NOTE]
> Always take a **backup** of the **PolicyDefensions** folder before adding new or replacing ADMX and ADML files.

After you extract the templates, open the \policy_templates\windows folder. Copy the Chrome admx files, to `%systemroot%\PolicyDefinitions`.

Here you see the admx files has copied to the local client machine in the source location  `C:\Windows\PolicyDefinitions` folder.

![[Pasted image 20240223093418.png]]

Once you have copied the ADMX file, you have to copy the ADML files. In the admx folder, open the appropriate language folder. **For example,** open the en-US folder if you’re in the U.S.

Copy the Adml files from the `windows\admx\en-US` folder path to `%systemroot%\PolicyDefinitions\en-US`.

Here you see the adml files has copied to the local client machine in the source location `C:\Windows\PolicyDefinitions` **\en-US** folder.

![[Pasted image 20240223093443.png]]

Once the files are successfully placed, You can open **Local Group Policy Editor** (Gpedit.msc) and see all the updated lists of policies for Google Chrome added in `Computer\User Configuration`.

![[Pasted image 20240223093546.png]]