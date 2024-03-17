---
created: 2024-02-28T17:05
updated: 2024-02-28T17:08
tags:
  - Proxmox
related link: https://pve.proxmox.com/wiki/Renaming_a_PVE_node
---
## Introduction

Proxmox VE uses the _hostname_ as a nodes name, so changing it works similar to changing the host name. This must be done on a empty node.

## Change Hostname

Rename a standalone PVE host, you **need** to edit the following files:

- /etc/hosts
- /etc/hostname

There are other files which you may want to edit, they are not important for the functions of Proxmox VE itself.

- /etc/mailname
- /etc/postfix/main.cf

If your node is in a **cluster**, where it is not recommended changing its name, adapt `/etc/pve/corosync.conf` so that the nodes name is changed also there. See [https://pve.proxmox.com/pve-docs/chapter-pvecm.html#pvecm_edit_corosync_conf](https://pve.proxmox.com/pve-docs/chapter-pvecm.html#pvecm_edit_corosync_conf)

## Cleanup

The SSH keys don't need to be edited unless you really want to (and if you do, make sure you make the corresponding change on every other machine that SSH public key appears on).

> [!NOTE]
> Now move the configuration files, as the _pmxcfs_ has a few restrictions to ensure consistency you cannot rename non empty folders. 

- If you have VMs or Containers on the node, which is not recommended when changing a nodes name, you have to recreate the folder structure and copy files per folder level.

- Copy the contents of `/var/lib/rrdcached/db/pve2-{node,storage}/old-hostname` to `/var/lib/rrdcached/db/pve2-{node,storage}/new-hostname` and remove the old directory.

