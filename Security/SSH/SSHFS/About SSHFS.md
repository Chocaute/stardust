---
created: 2024-04-02T08:39
updated: 2024-04-02T10:14
tags:
  - SSH
  - Filesystems
---


To mount a filesystem from a remote host through an intermediary:
```shell
sshfs ma: /mnt -o ssh_command='ssh -J mb'
```

Exemples:
```
sshfs root@10.10.0.100:/opt/ /home/tom/SSHFS/121 -o ssh_command='ssh -J charlie@proxmox.nuc.home.tomraud.fr'
```

What's useful about this command is that you don't need to know the password or have a pubkey installed on the intermediary. 

### Further reading

[NAS Performance: NFS vs. SMB vs. SSHFS](https://blog.ja-ke.tech/2019/08/27/nas-performance-sshfs-nfs-smb.html)

