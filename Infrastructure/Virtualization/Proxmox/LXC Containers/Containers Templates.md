---
created: 2024-02-28T17:09
updated: 2024-02-28T17:57
tags:
  - Proxmox
  - Containers
related link: https://pve.proxmox.com/wiki/Linux_Container#pct_container_images
---
## Container Images

Container images, sometimes also referred to as “templates” or “appliances”, are tar archives which contain everything to run a container.

Proxmox VE itself provides a variety of basic templates for the [most common Linux distributions](https://pve.proxmox.com/wiki/Linux_Container#pct_supported_distributions). They can be downloaded using the GUI or the pveam (short for Proxmox VE Appliance Manager) command-line utility. Additionally, [TurnKey Linux](https://www.turnkeylinux.org/) container templates are also available to download.

The list of available templates is updated daily through the _pve-daily-update_ timer. 

You can also trigger an update manually by executing:
```shell
pveam update
```
S
To view the list of available images run:
```shell
pveam available
```

You can restrict this large list by specifying the section you are interested in, for example basic system images:
```shell
List available system images

# pveam available --section system
system          alpine-3.12-default_20200823_amd64.tar.xz
system          alpine-3.13-default_20210419_amd64.tar.xz
system          alpine-3.14-default_20210623_amd64.tar.xz
system          archlinux-base_20210420-1_amd64.tar.gz
system          centos-7-default_20190926_amd64.tar.xz
system          centos-8-default_20201210_amd64.tar.xz
system          debian-9.0-standard_9.7-1_amd64.tar.gz
system          debian-10-standard_10.7-1_amd64.tar.gz
system          devuan-3.0-standard_3.0_amd64.tar.gz
system          fedora-33-default_20201115_amd64.tar.xz
system          fedora-34-default_20210427_amd64.tar.xz
system          gentoo-current-default_20200310_amd64.tar.xz
system          opensuse-15.2-default_20200824_amd64.tar.xz
system          ubuntu-16.04-standard_16.04.5-1_amd64.tar.gz
system          ubuntu-18.04-standard_18.04.1-1_amd64.tar.gz
system          ubuntu-20.04-standard_20.04-1_amd64.tar.gz
system          ubuntu-20.10-standard_20.10-1_amd64.tar.gz
system          ubuntu-21.04-standard_21.04-1_amd64.tar.gz
```

Before you can use such a template, you need to download them into one of your storages. If you’re unsure to which one, you can simply use the local named storage for that purpose. For clustered installations, it is preferred to use a shared storage so that all nodes can access those images.

```shell
pveam download local debian-10.0-standard_10.0-1_amd64.tar.gz
```

You are now ready to create containers using that image, and you can list all downloaded images on storage local with:
```shell
pveam list local
local:vztmpl/debian-10.0-standard_10.0-1_amd64.tar.gz  219.95MB
```

> [!NOTE] 
> You can also use the Proxmox VE web interface GUI to download, list and delete container templates.

pct uses them to create a new container, for example:

```shell
pct create 999 local:vztmpl/debian-10.0-standard_10.0-1_amd64.tar.gz
```

The above command shows you the full Proxmox VE volume identifiers. They include the storage name, and most other Proxmox VE commands can use them. 

For example you can delete that image later with:
```shell
pveam remove local:vztmpl/debian-10.0-standard_10.0-1_amd64.tar.gz
```
