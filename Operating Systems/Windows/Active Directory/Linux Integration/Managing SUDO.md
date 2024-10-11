---
created: 2024-02-22T11:18
updated: 2024-03-11T19:57
tags:
  - AD
  - Linux
  - Windows
  - sudo
related link: https://www.sudo.ws/docs/man/sudoers.ldap.man/
---
# The basics
https://michaelwaterman.nl/2022/10/21/managing-sudo-from-active-directory/

> Configure the domain and Ubuntu desktop in such a way that members of a domain security group will be able to use sudo once logged onto the Ubuntu desktop.

### Make Active Directory capable for supporting centralized sudo management.

Out of the box AD can’t be used for managing sudo users because the attributes aren’t available in the AD schema.

```powershell
wget https://raw.githubusercontent.com/sudo-project/sudo/main/docs/schema.ActiveDirectory -O schema.ActiveDirectory
```

> [!NOTE]
> This file is used to extent the Active Directory Schema to include the sudo attributes that we will be using.

Execute the following `ldifde` command, leave the command-line exactly as is, only replace the latter part of the line to reflect your domain structure.

```powershell
ldifde.exe -i -f schema.ActiveDirectory -c dc=X,dc=water,dc=lab
```

```powershell
Connecting to "LAB-DC01.water.lab"
Logging in as current user using SSPI
Importing directory from file "schema.ActiveDirectory"
Loading entries.............
12 entries modified successfully.

The command has completed successfully
```

### Add the "sudoRole" object. 

1. Next, open “_adsiedit.msc_” and connect to the “_Default naming context_“.

2. In the root create a new organization unit and give it a name “_sudoers_“. 

> [!NOTE]
> This is the default location sudo will look for user defined rules in AD, don’t worry, I’ll show you later on how you can change that.

3. On the organizational unit that you just created, right click and select “_new-object_“. In the next window select “_sudoRule_”.

> [!NOTE]
> If you don’t see the object class immediately that probably means that the class hasn’t been replicated to your DC yet, just give it some time.

4. In the next screen, use the name “_default_”. 

> [!NOTE]
> In my experience it really doesn’t matter what name you give the object. I tend to use a more descriptive name like, desktops or servers. That makes it a bit more clear on what purpose the object serves.

5. At the last screen, click “_Finish_” to create the “_sudoRole_” object.

> [!NOTE]
> Now we need to edit the attributes of the default object we just created. This way we can configure the behavior of the sudo commands into what can (or cannot) be executed on the Linux host.
> 
> Just for testing purposes, we’re going to enable all sudo privileges on all machines.

6. Select properties on the “_cn=default,ou=sudoers_” object. Edit these attributes:

- sudoCommand: ALL
- sudoHost: ALL
- sudoUser: ALL

![[Pasted image 20240222113423.png]]

### Configuring the Ubuntu client.

> [!NOTE]
> Ubuntu 22.0.4 and above have a enhanced way of applying management and configuring from the domain involving a technology alike Windows with Group policies. 
> This configuration is based on ADMX files, and at the time of writing this tech isn’t fully released yet. 
> We rely heavily on this new technology and lacks a critical package and configuration for applying sudoer from the domain using the procedure described here. 

Add the full sudoers support to the machine. Open a shell and type:
```
apt install sudo libsss-sudo -y
```

Add the following to the “_/etc/nsswitch.conf_“, just underneath the line: “_netgroup nis sss_“:
```
sudoers:	files sss
```

Add the following to the “etc/sssd/sssd.conf“, under the “_[sssd]_” section:
```
service = nss, pam, sudo
```

Restart the SSSD deamon.
```
sudo systemctl restart sssd
```

Test this setup with a domain admin:
```
sudo -l -U darth
```
![[Pasted image 20240222114001.png]]

### Domain Security Groups

> [!NOTE]
> In a managed infrastructure of some size it’s highly unlikely that you'll have many individual users that will be listed on the “sudoRole” object. Having a group listed on it makes more sense. 
> Instead of a user that can be set by just using the `sAMAccountName`, a group will need to contain a % sign at the beginning. 

So suppose we want to copy the behavior of Windows and add the “Domain Admins” to every machine (Which is a terrible idea!):
![[Pasted image 20240222114245.png]]

> [!NOTE]
> Although you would expect there to be an escape character () after “%Domain” to cope with the space, this seems to be completely handled by the sssd service.
> 
> There’s also a possibility to use wildcard characters for the attributes (See sudoHost). There are more options available listed here: [Sudoers Manual | Sudo](https://www.sudo.ws/docs/man/sudoers.man/#Wildcards)

### Organizational Units

> [!NOTE]
> By default sudo will look for an organizational unit with the name “_sudoers_” in the root of the directory. There is however a way to tell sudo to look elsewhere. 
> For example, if you have two OU’s assigned to different groups, you could use that setup to apply different sudo configurations. 

![[Pasted image 20240222114515.png]]

The Ubuntu machine in our example belongs to that OU and we want to point it specifically to a “_sudoRules_” object.

On the Ubuntu host, open the file “_etc/sssd/sssd.conf_”. Add the following line under the “_[domain/YourDomain]_“:
```
ldap_sudo_search_base = ou=the,ou=sudoRole,ou=OU,dc=water,dc=lab
```

Then reboot your linux machine.

### Troubleshooting Tips

> [!NOTE]
> Sometimes the caching mechanism from the sssd service is so persistent that the entries for sudo users remain, even after several reboot. 

One command in particular has helped me to clear the cache is:
```
sss_cache -E
```

If that doesn’t work, stop the sssd service and delete the database files in `/var/lib/sss/db_*`: 
```
rm /var/lib/sss/db/*
```

Just remember that the cache receives a full update every six hours and an incremental one every 15 minutes. But that really doesn’t help when the cache is broken. Those values can be altered by setting:

- ldap_sudo_full_refresh_interval=86400
- ldap_sudo_smart_refresh_interval=3600


# More in-depth sudo roles management
https://www.haxor.no/en/article/sudo-with-ad

> To make it as easy to manage the SUDO roles as possible I decided to create a new OU at the root of the domain directory. This was done by launching the ADSI Editor (Active Directory Interface Editor) from Server Manager.
> 
> The ADSI editor is a more fully-featured LDAP object editor that makes it possible to manage LDAP objects that are not managed as frequently as User, security group, and Group Policy Objects.

### On the Active Directory

Connect to the Domain Controller and create a new OU by right-clicking on the top OU and selecting New > Object... from the contextual menu.
![[Pasted image 20240222115956.png]]

From the list of classes, choose the organizationalUnit class, and give it an appropriate name.
![[Pasted image 20240222120018.png]]

I gave my OU the name haxorSudo to adhere to my own naming convention on the Domain.

Next, a SUDO role object could be added to the OU. This is done by right-clicking on the haxorSudu OU that was just created and selecting New > Object... from the contextual menu.
![[Pasted image 20240222120044.png]]

Now, select the sudoRole class from the list, and give it a name that makes it easy to distinguish from other SUDO roles.
![[Pasted image 20240222120104.png]]

In this example I gave it the name "webDeveloper - restart Nginx" to reflect that this SUDO role gives members of the webDeveloper security group permissions to restart Nginx.

To define the rules, right-click on the SUDO role object, and select Properties from the contextual menu.

Scroll down to the attributes that start with "sudo" to set the needed values.
![[Pasted image 20240222120120.png]]
*Add SUDO role parameters to the SUDO role object.*

> [!NOTE]
> If you are familiar with the visudo command or more specifically the syntax in the /etc/sudoers file on Linux hosts, these parameters should be somewhat intuitive. 
> Each SUDO role object corresponds to a line in the sudoers file. So you want to add several rules, you must add several SUDO role objects to AD.

In the example SUDO role above, I limited the rule to only apply to the host `www.haxor.local`, which is the webserver on my lab domain. I also limited the role to only allow the members of webDevelopers security group to be able to run "systemctl restart nginx" command as the root user. By doing this the SUDO role makes use of the rule of least privilege.

The LDAP object is similar to this line in a sudoers file:
```/etc/sudoers
%webdevelopers@haxor.local	www.haxor.local=(root) /bin/systemctl restart nginx
```
*Note: SSSD will translate the webDevelopers security group in AD to `webdevelopers@haxor.local` on the Linux hosts.*

### On the linux host

To apply the changes restart SSSD:
```bash
systemctl restart sssd
```

Now you can sign in with a domain account that is a part of a SUDO-enabled user group in AD, and verify the configuration.
```bash
sudo -l
```

```shell
User eva@haxor.local may run the following commands on www:
(root) /bin/systemctl restart nginx
```

The output above confirms that the user eve@haxor.local, which is a part of the user group webdevelopers@haxor.local is able to access the SUDO role defined on the AD DC.

#### Clear cache to be able to get recent changes to SUDO roles

> [!NOTE]
> Since users, user groups, and other data from the domain controllers are cached by SSSD you will probably need to clear the cache to quickly see the changes that affect users on your Linux hosts.

To clear the cache simply stop SSSD, delete the cached files, and start SSSD.

```bash
systemctl stop sssd
rm -rf /var/lib/sss/db/*
systemctl start sssd
```

If you now authenticate as a domain user on the Linux host, the changes will be synced.