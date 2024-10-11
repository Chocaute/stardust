---
created: 2024-03-12T15:44
updated: 2024-03-12T15:46
tags:
  - Linux
  - Debian
  - Bootstraps
related link: https://wiki.debian.org/Debootstrap
---
> [debootstrap](https://packages.debian.org/debootstrap "DebianPkg") is a tool which will install a Debian base system into a subdirectory of another, already installed system. It doesn't require an installation CD, just access to a Debian repo.
>  
> It can also be installed and run from another operating system, so, for instance, you can use debootstrap to install Debian onto an unused partition from a running Gentoo system. 
> 
> It can also be used to create a rootfs for a machine of a different architecture, which is known as "cross-debootstrapping". 

To setup a stable system:
```shell
mkdir /stable-chroot
debootstrap stable /stable-chroot http://deb.debian.org/debian/
```

To setup an Ubuntu system from Debian:
```shell
mkdir /ubuntu_xenial_1604
debootstrap --arch=amd64 xenial /ubuntu_xenial_1604 http://archive.ubuntu.com/ubuntu/
```