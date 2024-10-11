---
created: 2024-02-22T11:04
updated: 2024-02-22T11:10
tags:
  - Windows
  - AD
  - Linux
  - GPO
---
https://michaelwaterman.nl/2022/10/07/advanced-ubuntu-configuration-with-active-directory/

# The basics 

On a Windows machine, open up “_Group Policy Management_“, create a new policy with the scope of management applied to the Ubuntu machine. Open the policy for editing and go to:

“_Computer Configuration – Policies – Windows Settings – Security Settings – Local Policies – User Right Assignment_“

- Allow logon locally: “Administrators”, “Water\Domain Admins”
- Deny logon locally: “Water\Domain Users”

Reboot your machine or restart the sssd daemon. Try to logon with a regular user account, this will fail, however a subsequential logon with a domain admin account will be granted.
___
# Controlling Group Policy processing

The processing of group policies can be configured in the `/etc/sssd/sssd.conf` file. 
Sometimes it could be beneficial to not have policies being processed for troubleshooting purposes, but it could be in violation with your companies policy. 
This is also another reminder why you should avoid handing out sudo privileges to users as they could potentially block the required configuration, but that’s another story. 

Enter the following commands to open the file, enter the correct settings and block gpo processing:
```
sudo nano /etc/sssd/sssd.conf
```

Enter the following under [domain\domainname] section:
```
ad_gpo_access_control = disabled
```

Restart the sssd daemon or reboot and policies will no longer process. 

Potential other values for the key “_ad_gpo_access_control_” are:

|Value|Description|
|---|---|
|disabled|Processing of Group Policies is disabled|
|enforcing|Processing of Group Policies is enabled and forcefully applied to the system|
|permissive|Processing of Group Policies is being audited. Deny messages will be logged.|

According to my findings, but I didn’t test them all, the following Group Policy Security settings will by default be processed:

- Allow log on locally
- Deny log on locally
- Allow log on through Remote Desktop Services
- Deny log on through Remote Desktop Services
- Access this computer from the network
- Deny access to this computer from the network
- Allow log on as a batch job
- Deny log on as a batch job
- Allow log on as a service
- Deny log on as a service
___

