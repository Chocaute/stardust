---
created: 2024-04-02T10:29
updated: 2024-04-02T10:40
tags:
  - AD
  - Proxmox
  - LXC
related link: https://notes.benheater.com/books/linux-administration/page/proxmox-unprivileged-lxc-workaround
---
### About

Upon joining a host to the Active Directory domain, it was impossible to SSH in as one of my domain users. This is due to the misaligned **_uid_** and **_gid_** mapping between Proxmox and the high _**uid**_ and **_gid_** used by `sssd`.

Referencing the Proxmox VE documentation: https://pve.proxmox.com/wiki/Unprivileged_LXC_containers

You'll note that when a Linux Container is unprivileged, `root` in the container `uid: 0` and `gid: 0` is mapped to a high `uid` and `gid` on the host, so that if a container escape were to occur, it would only yield a shell in an privileged user session.

### SSSD High UID and GID Mapping

If you look in `/etc/subuid` and `/etc/subgid` on the PVE host, you'll note the syntax of:

```
root:100000:65536
```

Referencing `man subuid`, you'll note that this indicates that _**starting from**_ UID / GID 100000, there can be 65,536 mappings for the _**root**_ user; effectively `100000 -- 165535`. This is defined for `root`, as this is the default user when a new Linux Container is created.

### The Problem

![[Pasted image 20240402103356.png]]

The problem here is that the _**uid**_ and _**gid**_ mappings used by `sssd` when syncing domain users and groups are higher than the `1000000` id in the original config.

### The Solution

We should allow a higher subordinate user and group ID mapping, so that the domain-joined Linux Container can map the user IDs appropriately and allow logins.

> [!warning] Note
>  In the screenshot, the output of the command `id domain.admin@ad.lab` indicates the `uid` of `519801105` and the `gid` of `519800513`, `519800512`, `519800572` for this particular user.  
>   
> I'm using the `root:500000000:99999999` and not a much higher range like you might see [here, for example:](https://www.reddit.com/r/Proxmox/comments/19em9ud/sshd_issues_with_ad_domain_users/).

> [!tip] About the above note
> _**There's nothing wrong with using a high value as shown in this example**_. I simply didn't need it, because the range I chose is sufficienlty large enough to cover the `uid`and `gid` output as shown in the `id domain.admin@ad.lab` command.   
>   
> The `uid` and `gid` ranges are going to vary between Active Directory environments. So, before you apply the solution, understand the necessary range as required by your environment by inspecting users and groups using the `id` command on your Linux

Run these commands **_on the PVE node_**. The syntax `50000000:9999999` indicates that we'll have a _**range**_ of available for user and group IDs spanning `50000000 -- 59999998`.

```shell
cp /etc/subuid /tmp/subuid.bak
cp /etc/subgid /tmp/subgid.bak
echo 'root:500000000:99999999' >> /etc/subuid
echo 'root:500000000:99999999' >> /etc/subgid
```

One more piece to the puzzle yet... We need to tell the Linux Container that it should use this mapping, while still continuing to use its original mappings for the `root` user.

```shell
container_id=184
cp "/etc/pve/lxc/$container_id.conf" "/tmp/$container_id.conf.bak"
echo 'lxc.idmap = u 0 100000 65536' >> "/etc/pve/lxc/$container_id.conf"
echo 'lxc.idmap = g 0 100000 65536' >> "/etc/pve/lxc/$container_id.conf"
echo 'lxc.idmap = u 500000000 500000000 99999999' >> "/etc/pve/lxc/$container_id.conf"
echo 'lxc.idmap = g 500000000 500000000 99999999' >> "/etc/pve/lxc/$container_id.conf"
pct reboot $container_id
```

The `u 500000000 500000000 99999999` and `g 500000000 500000000 99999999` syntax indicates that any **u**ser or **g**roup starting with the ID of `500000000` should begin using the mappings. 

___

If everything is working, clean up your backed up files:

```shell
rm /tmp/subuid.bak
rm /tmp/subgid.bak
rm "/tmp/$container_id.conf.bak"
```