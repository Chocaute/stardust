---
created: 2024-02-22T09:35
updated: 2024-02-24T22:57
tags:
  - Windows
  - Linux
  - AD
  - SSSD
  - REALMD
---
# Prerequisites
https://michaelwaterman.nl/2022/10/02/domain-join-ubuntu-22-04-to-active-directory/
https://michaelwaterman.nl/2022/10/07/advanced-ubuntu-configuration-with-active-directory/

- First thing that we need to check before joining the machine is the hostname.

Although Ubuntu can handle long hostnames, a Windows machine is restricted to 15 characters max because of the legacy protocol NetBios. 
In turn that means that our Ubuntu client machines is also restricted to the same length for it’s hostname. 

Check the current hostname with this command:
```
sudo hostnamectl
```

In /etc/hostname:
```
your-hostname
```

In /etc/hosts:
```
127.0.0.1    localhost
127.0.1.1    your-hostname.your-domain.your-tld    your-hostname
```

- Second, the machine DNS address needs to be the AD's. 
- Third, timing is everything. Ensure the machine and the AD are set to use the same timezone.

- If you want to create a new home folder upon new user login:
```
sudo pam-auth-update --enable mkhomedir
```

- Set the properties for the Active Directory Computer object:
```
touch /etc/realmd.conf && nano /etc/realmd.conf
```

```
[active-directory]
default-client = sssd
os-name = Ubuntu Workstation
os-version = 22.04
```

> [!NOTE]
> This is obviously just a small fraction on how you can influence the domain join process and the configuration of the SSSD daemon. The file itself uses a very simple construct of a key-value pair divided into sections, much like ini files which are often used on Windows based operating systems. I’ve listed the frequently used settings in the table below.

|Section:  <br>[active-directory]|Default value|realmd.conf property|Description|
|---|---|---|---|
|default-client|_sssd_ or _winbind_|[sssd] section|Sets the default client software package for joining and managing the domain activities.|
|os-name|pc-linux-gnu|None|Sets the Active Directory property “_OperatingSystem_” of the computer object.|
|os-version|None|None|Sets the Active Directory property “_OperatingSystemVersion_” of the computer object.|

|Section: [service]|Default value|realmd.conf property|Description|
|---|---|---|---|
|automatic-install|No|None|automatic installation of required and missing packages using “_package-kit_“. Note that “_package-kit_” itself needs to be installed first.|

|[realmname]|Default value|realmd.conf property|Description|
|---|---|---|---|
|automatic-id-mapping|On|ldap_id_mapping|leave this value set to default if you don’t have the POSIX attributes set in Active Directory.|
|automatic-join|Off|None|Automatically joins a machine to active directory if a computer object already exists.|
|computer-ou|Off|None|Sets the location of the computer object in Active Directory, use DN notation. E.G. “_OU=Ubuntu,_  <br>_DC=water,DC=lab_“|
|computer-name|Off|None|Define the name of the computer object. Leave this value to default as it can lead to misconfigurations. Only set for in exceptional circumstances.|
|fully-qualified-names|On|use_fully_qualified_names|Determines if a user logon needs to be done with the UPN or just the username. Default is UPN.|
|manage-system|On|realmd_tags|Influences management from the domain. Recommended to leave this enabled, which is the default.|
|user-prinicpal|On|None|Creates a service principle name (SPN) on the computer object|

To see the UPN being applied, use the following PowerShell command on a Domain joined Window device:

```
Get-ADComputer -filter * | Sort-Object -Property Name | Where Name -EQ "ubuntu" | Format-Table -Property Name, UserPrincipalName
```

The result should look like this:
![[Pasted image 20240222110341.png]]
___
# Installation and joint
https://www.server-world.info/en/note?os=Ubuntu_22.04&p=realmd

```
apt -y install realmd sssd sssd-tools libnss-sss libpam-sss libsss-sudo adcli samba-common-bin oddjob oddjob-mkhomedir packagekit
```

Discover Active Directory domain :
```
realm discover SRV.WORLD
```

Results:
```
srv.world
  type: kerberos
  realm-name: SRV.WORLD
  domain-name: srv.world
  configured: no
  server-software: active-directory
  client-software: sssd
  required-package: sssd-tools
  required-package: sssd
  required-package: libnss-sss
  required-package: libpam-sss
  required-package: adcli
  required-package: samba-common-bin
```

Join in Active Directory domain:
```shell
realm join SRV.WORLD
Password for Administrator:  
```

Verify it's possible to get an AD user info or not:
```shell
id Serverworld@srv.world
```

Results:
```
uid=1259201103(serverworld@srv.world) gid=1259200513(domain users@srv.world) groups=1259200513(domain users@srv.world),1259200512(domain admins@srv.world),1259200572(denied rodc password replication group@srv.world)
```

Only use the name of the user instead of the entire “_User Principal Name_”:
```shell
nano /etc/sssd/sssd.conf
```

In [domain/your-domain]
```shell
use_fully_qualified_names = False
```

```shell
systemctl restart sssd
```

