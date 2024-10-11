---
created: 2023-12-15T15:41
updated: 2023-12-15T20:54
---
#Linux #FS #LVM #LVMthin #NoZeroing

Ensure you have a backup of the data on your Btrfs partition before proceeding.

Unmount the Btrfs filesystem on `/dev/nvme0n1p1`.
```
umount /dev/nvme0n1p1
```

Use a partitioning tool (e.g., `gdisk`, `parted`) to delete the Btrfs partition `/dev/nvme0n1p1`.

Use the same partitioning tool to create a new partition on `/dev/nvme0n1`. This new partition will be used for the LVM setup.
 - Be careful not to overwrite the entire SSD (`/dev/nvme0n1`) as this would erase all data on the drive.

Create a new physical volume on the new partition.
```
pvcreate /dev/nvme0n1p1
```

Create a new volume group and a thin pool.
```
vgcreate pve_nvme /dev/nvme0n1p1
lvcreate --type thin-pool -l 500G -n data pve_nvme -Zn
lvdisplay -m /dev/pve_nvme/data 
```

Create a thin logical volume within the thin pool.
```
lvcreate --type thin -V 300G --name local_nvme pve_nvme/data
```

Format the thin logical volume with XFS.
```
mkfs.xfs /dev/pve_nvme/local_nvme
```

Create a mount point.
```
mkdir /mnt/pve/local_nvme
```

Mount the XFS-formatted logical volume.
```
mount /dev/pve_nvme/local_nvme /mnt/pve/local_nvme
```

**Restore the Data:**
Always ensure that you are working with the correct partition and that you have backups before making any changes. Additionally, be cautious when using partitioning tools to avoid accidental data loss.