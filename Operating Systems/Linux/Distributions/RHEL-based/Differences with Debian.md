---
created: 2024-03-12T16:24
updated: 2024-03-12T16:25
tags:
  - RHEL
  - Linux
---
> In a simple explanation, everything. They're 2 entirely different distributions with 2 entirely different goals from 2 entirely different companies that just so happen to both leverage the Linux kernel and default to the Gnome desktop.

Quelques extraits de conversation.

- RHEL utilise le système de gestion de paquets RPM (Red Hat Package Manager).
- Sur RHEL, l’équivalent est la commande yum ou dnf, qui utilise le système de gestion de paquets RPM.

Je me souviens d'un comportement différent des gestions de packets. yum et dns ne démarre pas packets directement après installation.

Bing CHAT m'a dit ça entre autres:


- How to use the subscription-manager command to register your RHEL system and manage your subscriptions.

- How to use the SELinux security framework to enforce mandatory access control policies on RHEL, which is not enabled by default on debian2.

![[selinux-context.png]]

- How to use the firewalld service to configure and manage firewall rules on RHEL, which is different from the iptables or ufw commands on debian2.

	- Firewalld is newer and more user-friendly, but it has less fine-grained control and it may not be compatible with some applications or scripts that rely on iptables.
	- Firewalld uses zones and services to group network interfaces and ports into different security levels.

- How to use the nmcli or nmtui commands to configure and manage network connections on RHEL, which are different from the ifconfig or ip commands on debian2.

	- I can use the /etc/sysconfig/network-scripts/ifcfg-(interfacename) file to configure network interfaces manually.

- How to use the systemctl or journalctl commands to manage and troubleshoot system services and logs on RHEL, which are similar to the ones on debian but have some differences in syntax and options2.

- How to use the rpm or rpm-ostree commands to query and verify software packages on RHEL, which are different from the dpkg command on debian12.

- How to use the podman, buildah, or skopeo commands to work with containers on RHEL, which are different from the docker command on debian12.

- How to use the ansible, puppet, or chef commands to automate system administration tasks on RHEL, which are similar to the ones on debian but may have some differences in configuration and modules.
