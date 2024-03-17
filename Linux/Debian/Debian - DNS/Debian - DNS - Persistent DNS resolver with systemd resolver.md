---
created: 2024-03-13T16:37
updated: 2024-03-13T17:08
tags:
  - Debian
  - DNS
related link: https://sleeplessbeastie.eu/2022/08/24/how-to-configure-persistent-dns-resolver/
---
> Configure persistent DNS resolver using systemd resolver or name server information handler.
### systemd resolver

Inspect status of `systemd-resolved` service:
```shell
systemctl status systemd-resolved.service 

● systemd-resolved.service - Network Name Resolution
     Loaded: loaded (/lib/systemd/system/systemd-resolved.service; disabled; vendor preset: enabled)
     Active: inactive (dead)
       Docs: man:systemd-resolved.service(8)
             man:org.freedesktop.resolve1(5)
             https://www.freedesktop.org/wiki/Software/systemd/writing-network-configuration-managers
             https://www.freedesktop.org/wiki/Software/systemd/writing-resolver-clients
```

Update service configuration:
```shell
sudoedit /etc/systemd/resolved.conf 
```


```shell
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.
#
# Entries in this file show the compile time defaults.
# You can change settings by editing this file.
# Defaults can be restored by simply deleting this file.
#
# See resolved.conf(5) for details

[Resolve]
# Some examples of DNS servers which may be used for DNS= and FallbackDNS=:
# Cloudflare: 1.1.1.1 1.0.0.1 2606:4700:4700::1111 2606:4700:4700::1001
# Google:     8.8.8.8 8.8.4.4 2001:4860:4860::8888 2001:4860:4860::8844
# Quad9:      9.9.9.9 2620:fe::fe
**DNS=10.10.0.1</code>**
#FallbackDNS=
**Domains=example.org**
#DNSSEC=no
#DNSOverTLS=no
#MulticastDNS=yes
#LLMNR=yes
#Cache=yes
#DNSStubListener=yes
#DNSStubListenerExtra=
#ReadEtcHosts=yes
#ResolveUnicastSingleLabel=no
```

Start and enable `systemd-resolved` service:
```shell
systemctl enable --now systemd-resolved.service
```

```shell
Created symlink /etc/systemd/system/dbus-org.freedesktop.resolve1.service → /lib/systemd/system/systemd-resolved.service.
Created symlink /etc/systemd/system/multi-user.target.wants/systemd-resolved.service → /lib/systemd/system/systemd-resolved.service.
```

Inspect service status:
```shell
status  systemd-resolved.service 
```

```shell
● systemd-resolved.service - Network Name Resolution
     Loaded: loaded (/lib/systemd/system/systemd-resolved.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2021-09-27 22:43:43 CEST; 16s ago
       Docs: man:systemd-resolved.service(8)
             man:org.freedesktop.resolve1(5)
             https://www.freedesktop.org/wiki/Software/systemd/writing-network-configuration-managers
             https://www.freedesktop.org/wiki/Software/systemd/writing-resolver-clients
   Main PID: 1093 (systemd-resolve)
     Status: "Processing requests..."
      Tasks: 1 (limit: 1105)
     Memory: 4.1M
        CPU: 46ms
     CGroup: /system.slice/systemd-resolved.service
             └─1093 /lib/systemd/systemd-resolved
```

Inspect generated `resolv.conf` file:
```shell
cat /etc/resolv.conf
```

```shell
# This file is managed by man:systemd-resolved(8). Do not edit.
#
# This is a dynamic resolv.conf file for connecting local clients to the
# internal DNS stub resolver of systemd-resolved. This file lists all
# configured search domains.
#
# Run "resolvectl status" to see details about the uplink DNS servers
# currently in use.
#
# Third party programs should typically not access this file directly, but only
# through the symlink at /etc/resolv.conf. To manage man:resolv.conf(5) in a
# different way, replace this symlink by a static file or a different symlink.
#
# See man:systemd-resolved.service(8) for details about the supported modes of
# operation for /etc/resolv.conf.

nameserver 127.0.0.53
options edns0 trust-ad
search example.org
```

Use `resolvectl` utility display global and per-link DNS settings:
```resolvectl status

Global
         Protocols: +LLMNR +mDNS -DNSOverTLS DNSSEC=no/unsupported
  resolv.conf mode: stub
Current DNS Server: 10.10.0.1
       DNS Servers: 10.10.0.1
        DNS Domain: example.org

Link 2 (eth0)
Current Scopes: LLMNR/IPv4 LLMNR/IPv6
     Protocols: -DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
```

Use `resolvectl` utility to query the DNS resolver:
```shell
resolvectl query linux.org
```

```shell
linux.org: 2606:4700:3033::6815:eaa            -- link: eth0
           2606:4700:3031::ac43:a015           -- link: eth0
           104.21.14.170                       -- link: eth0
           172.67.160.21                       -- link: eth0

-- Information acquired via protocol DNS in 24.1ms.
-- Data is authenticated: no

### name server information handler

Name server information handler is provided by an additional package.
```
___
### resolvconf
```shell
apt info resolvconf
```

```shell
Package: resolvconf
Version: 1.87
Priority: optional
Section: net
Maintainer: resolvconf team <team+resolvconf@tracker.debian.org>
Installed-Size: 204 kB
Depends: lsb-base (>= 4.1+Debian3), debconf (>= 0.5) | debconf-2.0
Breaks: dhcp3-client (<< 4.1.1-P1-15+squeeze1), dnscache-run, sysv-rc (<< 2.88dsf-42)
Enhances: dhcpcd, dnsmasq, ifupdown, isc-dhcp-client, libc6, network-manager, nscd, pdnsd, ppp, pump, udhcpc
Homepage: http://alioth.debian.org/projects/resolvconf/
Tag: admin::configuring, interface::commandline, network::configuration,
 protocol::dns, role::program, use::configuring
Download-Size: 72.7 kB
APT-Sources: http://ftp.task.gda.pl/debian bullseye/main amd64 Packages
Description: name server information handler
 Resolvconf is a framework for keeping up to date the system's
 information about name servers. It sets itself up as the intermediary
 between programs that supply this information (such as ifup and
 ifdown, DHCP clients, the PPP daemon and local name servers) and
 programs that use this information (such as DNS caches and resolver
 libraries).
 .
 This package may require some manual configuration. Please
 read the README file for detailed instructions.
```

Install `resolvconf` package:
```shell
apt install resolvconf
```

Inspect configuration files:
```shell
s -l /etc/resolvconf/resolv.conf.d/
```

```shell
total 8
-rw-r--r-- 1 root root   0 Sep 28 00:07 base
-rw-r--r-- 1 root root 275 Sep 28 00:07 head
-rw-r--r-- 1 root root  43 Sep 28 00:05 original
-rw-r--r-- 1 root root   0 Sep 28 00:06 tail
```

The `original` file is just a backup, so you can restore the configuration at your discretion:
```shell
cat /etc/resolvconf/resolv.conf.d/original 
```

```shell
domain lan
search lan
nameserver 10.10.0.1
```

The other files are used to build `resolv.conf` configuration file. It is built using the `head` as file header, interface configuration (static or dhcp), `base` and `tail` at the end. You can create a link from the `tail` to the `original` to include it generated file:
```shell 
cat /etc/resolvconf/resolv.conf.d/head 
```

```shell
# Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8)
#     DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN
# 127.0.0.53 is the systemd-resolved stub resolver.
# run "resolvectl status" to see details about the actual nameservers.
```

Enable `resolvconf` at boot and run it now:
```shell
systemctl enable --now resolvconf
```

```shell
Synchronizing state of resolvconf.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable resolvconf
Created symlink /etc/systemd/system/sysinit.target.wants/resolvconf.service → /lib/systemd/system/resolvconf.service.
```

Inspect service status. It will update configuration and exit:
```shell
systemctl status resolvconf
```

```shell
● resolvconf.service - Nameserver information manager
     Loaded: loaded (/lib/systemd/system/resolvconf.service; enabled; vendor preset: enabled)
     Active: active (exited) since Tue 2021-09-28 00:36:52 CEST; 1min 23s ago
       Docs: man:resolvconf(8)
    Process: 641 ExecStart=/sbin/resolvconf --enable-updates (code=exited, status=0/SUCCESS)
   Main PID: 641 (code=exited, status=0/SUCCESS)
        CPU: 1ms
```

Inspect configuration generated for DNS resolver:
```shell
cat /etc/resolv.conf 
```

```shell
# Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8)
#     DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN
# 127.0.0.53 is the systemd-resolved stub resolver.
# run "resolvectl status" to see details about the actual nameservers.

nameserver 10.10.0.1
search lan
```

Notice, this will be a link:
```shell
ls -l /etc/resolv.conf 
```

```shell
lrwxrwxrwx 1 root root 29 Sep 27 23:12 /etc/resolv.conf -> ../run/resolvconf/resolv.conf
```

Append additional DNS configuration to the interface if required. Notice, it requires `resolvconf` utility:
```shell
sudoedit /etc/network/interfaces.d/eth0
```

```shell
# The primary network interface
allow-hotplug eth0

## static address
#iface eth0 inet static
#address 10.10.1.9
#netmask 255.255.0.0
#gateway 10.10.0.1
#**dns-nameserver 10.10.0.2**
#**dns-search example.net**

## dynamic dhcp address
iface eth0 inet dhcp
**dns-nameserver 10.10.0.2**
**dns-search example.net**
```

Reboot the operating system.

Inspect configuration generated for DNS resolver. Configuration from DHCP and interfaces file got merged:
```shell
cat /etc/resolv.conf 
```

```shell
# Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8)
#     DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN
# 127.0.0.53 is the systemd-resolved stub resolver.
# run "resolvectl status" to see details about the actual nameservers.

nameserver 10.10.0.2
nameserver 10.10.0.1
search example.net lan
```

You can inspect the source data used to generate the above configuration:
```shell
cat /var/run/resolvconf/interface/eth0.inet 
```

```shell
search example.net
nameserver 10.10.0.2
```

```shell
cat /var/run/resolvconf/interface/eth0.dhclient 
```

```shell
domain lan
nameserver 10.10.0.1
```

You can override `resolvconf` DHCP client hook to disable using data from the DHCP agent, but it requires an operating system reboot:
```shell
echo "make_resolv_conf() { : ; }" | sudo tee /etc/dhcp/dhclient-enter-hooks.d/resolvconf-disable
```

Append additional configuration:
```shell
cat /etc/resolvconf/resolv.conf.d/base 
```

```shell
nameserver 8.8.8.8
```

Generate a new configuration for DNS resolver:
```shell
sudo resolvconf -u
```

Inspect configuration generated for DNS resolver:
```shell
cat /etc/resolv.conf 
```

```shell
# Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8)
#     DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN
# 127.0.0.53 is the systemd-resolved stub resolver.
# run "resolvectl status" to see details about the actual nameservers.

nameserver 10.10.0.2
nameserver 8.8.8.8
search example.net
```
### Additional notes

These utilities are not mutually exclusive, you can use _name server information handler_ to append additional configuration.