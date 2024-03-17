---
created: 2024-02-22T11:11
updated: 2024-02-22T11:17
tags:
  - Windows
  - Linux
  - AD
  - SSSD
  - REALMD
---
https://michaelwaterman.nl/2022/10/07/advanced-ubuntu-configuration-with-active-directory/

Sometimes things break, don’t work from the start or there’s a misconfiguration somewhere. Where do you start? That’s where you need to have a little knowledge on how to troubleshoot the technology. Below are a few tips and tricks that you can use to troubleshoot your configuration when connecting to Active Directory.

## Configuration Files
Relevant configuration files for troubleshooting.

|Filename|Description|
|---|---|
|/etc/krb5.conf|Kerberos configuration file. Usually this can be left to default.|
|/etc/sssd/sssd.conf|Main configuration file for realm configuration.|
|/etc/realmd.conf|Initial configuration file for sssd.|
|/etc/nsswitch.conf|Service configuration for hostnames, password processing etc.|

## Services

SSSD status
```
systemctl status sssd
```

## Networking

Get the local hostname
```
hostname -f
```
This should return the fqdn of the host.

Get the network status
```
resolvectl status
```

Basic name resolving
```
dig -t a lab-dc01.water.lab
```

Basic name resolving with specific DNS server
```
dig -t a lab-dc01.water.lab @192.168.66.2
```

Query domain SRV records
```
dig -t SRV _ldap._tcp.ad.water.lab
```

Disable Reverse name lookup, first:
```
sudo nano /etc/krb5.conf
```

Then add the following:
```
[libdefaults]
rdns = false
```

## SSSD Configuration

List sssd configuration
```
realm list
```

List sssd status
```
realm list | grep configured
```
The output should be: “_configured: kerberos-member_“

Check the configuration of SSSD
```
sudo sssctl config-check
```

## Information Retrieval

Retrieve user information from Active Directory
```
getent passwd darth@water.lab
```
![[Pasted image 20240222111529.png]]

Get user groups
```
Option 1:
groups darth@water.lab

Option 2:
id darth@water.lab
```

Advanced information
```
sudo sssctl user-checks darth
```

Show discovered Active Directory Servers
```
sudo sssctl domain-status water.lab --servers
```

Show active Active Directory servers
```
sudo sssctl domain-status water.lab --active-server
```

## Authentication

Lists a users kerberos tickets
```
klist
```

Delete kerberos tickets
```
kdestroy
```

Manually get a kerberos ticket
```
kinit darth@water.lab
```

Manually test a login
```
sudo login
```

Show cached user inforation
```
sudo sssctl user-show darth@water.lab
```

Expire the sssd cache
```
sudo sssctl cache-remove
```

## Debugging

Enable debug mode
```
sudo sssctl debug-level [1-9]
```
You can also set the debug level directly in the “_/etc/sssd/sssd.conf_” file. Log information will end up in the “_/var/log/sssd/_.log*” files.

Determine the session to troubleshoot
```
sudo sssctl analyze request list -v
```
For the analyze to work, the debug level should be set to 7 or higher.

Analyze a specific session
```
sssctl analyze request show 949
```

Live view of system events
```
sudo tail -f /var/log/syslog
```