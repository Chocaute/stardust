---
created: 2024-02-22T09:27
updated: 2024-02-22T09:31
tags:
  - Linux
  - Proxmox
  - Filesystems
---
https://forum.proxmox.com/threads/volume-sizes-exceeds-the-size-of-thin-pool-and-free-space-in-volume-group.102814/

This warns that your thin pool `pve/data` is currently over-provisioned, i.e. you have assigned more space to volumes that is actually available. As long as there is actual free space nothing is wrong, but once the disks start filling up, it could happen that the pool runs out of space, which is bad.  

```
root@pxmx:~# lvs
  LV                       VG  Attr       LSize    Pool Origin        Data%  Meta%  Move Log Cpy%Sync Convert
  cache1                   pve Vwi-aotz--   10.00g data               99.94                                
  cache2                   pve Vwi-aotz--   10.00g data               0.01                                  
  data                     pve twi-aotz-- <141.43g                    16.54  1.95                          
  root                     pve -wi-ao----   55.75g                                                          
  snap_vm-100-disk-1_Snap1 pve Vri---tz-k   16.00g data vm-100-disk-1                                      
  snap_vm-100-disk-1_Snap2 pve Vri---tz-k   16.00g data vm-100-disk-1                                      
  snap_vm-101-disk-0_Snap1 pve Vri---tz-k   20.00g data vm-101-disk-0                                      
  snap_vm-101-disk-0_Snap2 pve Vri---tz-k   20.00g data vm-101-disk-0                                      
  swap                     pve -wi-ao----    7.00g                                                          
  vm-100-disk-1            pve Vwi-aotz--   16.00g data               9.71                                  
  vm-100-state-Snap1       pve Vwi-a-tz--   <3.49g data               18.07                                
  vm-100-state-Snap2       pve Vwi-a-tz--   <3.49g data               13.01                                
  vm-101-disk-0            pve Vwi-aotz--   20.00g data               21.14                                
  vm-114-disk-0            pve Vwi-aotz--    8.00g data               21.16                                
  vm-124-disk-0            pve Vwi-a-tz--    6.00g data               67.80                                
  vm-124-disk-1            pve Vwi-a-tz--    4.00g data               1.63
```

The `LSize` column shows the provisioned space for each volume and the `Data%` column shows how much space is currently actually used. The most important thing to keep an eye out is the `Data%` of the thin pool `data` itself (currently `16.54`).  

```
root@pxmx:~# vgdisplay pve
  --- Volume group ---
  VG Name               pve
  System ID            
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  127
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                16
  Open LV               7
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <223.07 GiB
  PE Size               4.00 MiB
  Total PE              57105
  Alloc PE / Size       53010 / 207.07 GiB  <- Space allocated
  Free  PE / Size       4095 / <16.00 GiB  <- Space used
  VG UUID               ug5DjO-FyBB-O0Gm-rxLW-ibQC-pPrS-jTgo1i
root@pxmx:~# df -h
Filesystem                  Size  Used Avail Use% Mounted on
udev                        3.8G     0  3.8G   0% /dev
tmpfs                       777M  1.1M  776M   1% /run
/dev/mapper/pve-root         55G  6.3G   46G  13% /
tmpfs                       3.8G   46M  3.8G   2% /dev/shm
tmpfs                       5.0M     0  5.0M   0% /run/lock
/dev/sda2                   511M  328K  511M   1% /boot/efi
zstore                      3.5T  128K  3.5T   1% /zstore
zstore/media                3.6T   63G  3.5T   2% /opt/media
/dev/fuse                   128M   20K  128M   1% /etc/pve
tmpfs                       777M     0  777M   0% /run/user/0
```
