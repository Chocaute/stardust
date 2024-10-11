---
created: 2023-12-15T16:01
updated: 2024-05-17T16:43
tags:
  - Filesystems
  - Proxmox
  - Linux
  - FS
  - LVMthin
  - NoZeroing
  - BTRFS
  - ext4
  - XFS
  - ZFS
  - Benchmarks
related link: https://web.archive.org/web/20230330145140/https://www.assyoma.it/single-post/2015/02/02/zfs-btrfs-xfs-ext4-and-lvm-with-kvm-a-storage-performance-comparison
---

> [!WARNING] Contents may be out-of-date
> The contents of this file date back from 2017. As such, the conclusions of these tests may not reflect reality anymore. However, I quite like the methodology, so I keep it around.

# A storage performance comparison
by some italian idk

Virtual machines storage performance is a hot topic – after all, one of the main problem when virtualizing many OS instances is to correctly size the I/O subsystem, both in term of space and speed.

Of course performance is not the only thing to consider: another big role is played by flexibility and ease to use/configure. So the perfect storage subsystem is the right compromise between performance, flexibility and ease to use.

In a somewhat ironic manner, of the three requisites written above, performance is the most difficult thing to measure, as“I/O speed” is an ephemeral concept. It is impossible to speak about I/O performance without taking into account three main parameters: I/O block size, I/O per seconds (IOPS) and queue depth (the number of outstanding, concurrent block requests). This represent the first problem to correctly size your disk subsystem: you had to guess about the expected access pattern, and you better guess right.

Another pitfall is how to provision (allocate) space inside the configured storage subsystem: it is better to use Fat or Thin provisioning? What are the performance implications?

At the same time, flexibility and ease to use are easy things to sell: I often read of how modern, CoW filesystems as BTRFS and ZFS have plenty of features and of how many peoples recommend to use them for performance-sensitive tasks as virtual machines storage.

Sure BTRFS and ZFS have a host of slick features, but are they really suited for storing “live” virtual machines data, or our beloved legacy filesystems as EXT4 and XFS are better suited for this task? Or what about going a layer down, directly playing with Logical Volumes?

I'll try to answer these questions. Anyway, please keep in mind that the result of any benchmark is of relative values – I don't pretend to elect the uber-mega-super-duper I/O configuration. I only hope to help you selecting the right tool for the right job.

## Testbed and Methods

### Testbed

Benchmarks were performed on a system equipped with: 
- PhenomII 940 CPU (4 cores @ 3.0 GHz, 1.8 GHz Northbridge and 6 MB L3 cache) 
- 8 GB DDR2-800 DRAM (in unganged mode) 
- Asus M4A78 Pro motherboard (AMD 780G + SB700 chipset) 
- 4x 500 GB, 7200 RPM hard disks (1x WD RE, 3x Seagate Barracuda) in AHCI mode, configured in software RAID10 "near" layout (default 512K chunks) 
- S.O. CentOS 7.0 x64, kernel version `3.10.0-123.13.2.el7.x86_64`

All disk had 3 partitions, used for construct three MD arrays: 
- one first RAID1 “boot” array, 
- one RAID10 “system” one (/ + swap, via LVM),
- one final “data” RAID 10 (~ 800 GB usable) array for VMs hosting and testing. 

BTRFS and ZFS natively support mirroring + striping, so in these cases I go ahead without the “data” MD array and used the integrated facilities to create a mirror+striped dataset.

ZFS was used with the kernel-level driver provided by ZFS on Linux (ZoL) project, version 0.6.3-1.2. A slight change in default setting was needed for 100% reliable operation: I had to enable the `xattr=sa` option. [You can read more here.](https://web.archive.org/web/20230330145140/http://github.com/zfsonlinux/zfs/issues/1548)

### Methods

A benchmark run consisted in: 
1. Concurrently install the 4 VMs (using PXE on the host side + kickstart for unattended installation). The virtual machines had default configuration values, using LVM+XFS for guest systems. The four VM images were 8 GB each in size.
2. Prepare a PostgreSQL database on one VM, using sysbench on the host side (using default parameters); 3) concurrently runs a different benchmark on each VM for ten minutes (600 seconds), three time in a row; the results were averaged.

 Step **3.** needs some more explanations: 
 - The first machine runs PostgreSQL and was benchmarked by a sysbench instance running on the host. I used the “complex” test with default options. The “complex” test is a mixed read/write, transactional test; 
 - The second machine runs filebench with fileserver personality, to simulate fileserver-type activity. It is a mixed read/write, mixed sequential/random test; 
 - The third machine runs filebench with varmail personality, issuing many small synchronized writes as many mailserver do; 
 - The fourth machine runs filebench with webserver personality, issuing many small reads as typically webservers do.

#### Disclaimer

You can consider any synthetic or semi-synthetic test as flawed, and in some manner you are right. At the same time, the above tests are both easy reproducible (I used default options, both for configurations and benchmarks) and I/O heavy. Moreover, they represent and use a more-or-less credible I/O pattern.

You may wonder why I installed the 4 virtual machines concurrently, and why I run these very stressful tests all at the same time. **The fact is that thin provision has a hidden threat:** when issuing many I/O writes concurrently, data can be scattered all over the disk, leading to considerable fragmentation. Instead, when using a single VM at a time this behavior is usually not noticeably, as the system has no disk bandwidth contention in this case. The only exception was the database creation/population: it is a very specific operation that need to be issued to a single VM and in real world you will rarely load an entire DB during periods of high I/O usage.

One last thing: I did not specifically optimize guests configuration for maximum throughput. For example, the PostgreSQL service runs with default value for buffer size. For this benchmark round I was interested in creating a high I/O load, nor to fine-tune guests configuration.

### The configurations to test

I/O under Linux can be configured in a multitude of ways, even more when you factor in how libvirt/KVM supports high level file container as Qcow2, QED and the likes. 

So, while I like to give you a broad view of the various configurations, I can not test every options. I restricted myself to testing only a subset of possible storage configurations, and I explicitly excluded the options that do not supports snapshots. So, while attaching a direct MBR or GTP style partition to a VM can be fastest options (and sometime make sense) I did not consider this case here.

For the same reason, **I generally used the distribution's default parameters. Unless noted differently, I benchmarked both fat and thin provisioning configurations.**

The tested scenarios are:

1. **Qcow2 backend on top of XFS filesystem on top of a raw MD device**. Both thin and partial (metadata only) preallocation modes were benchmarked; 
   
2. **Logical Volumes backend, both in classical LVM (fat preallocation) and thin (thin lvm target) modes.** Moreover, thin lvm was analized with both zeroing on and off; 
   
3. **XFS and EXT4 on top of classical LVM**, relaying on filesystem sparse-file support for thin provisioning; 
   
4. **XFS and EXT4 on top of thin LVM**, relaying on thin lvm target for thin provisioning. In this case, LVM zeroing was disabled as the to-be-zero blocks are directly managed inside the filesystem structures; 
   
5. **BTRFS on top of its mirror+stripe implementation** (no MD here). I benchmarked BTRFS with CoW both enabled and disabled (nodatacow mount option) 
   
6. **ZFS on top of its mirror+stripe implementation** (no MD again)

Ok, lets see how things add up...

## Tests

### Virtual Machines installation

The first benchmark consisted in a timed, concurrent installation of four virtual machines. I used PXE to boot the VMs, a `CentOS-7.0-1406-x86_64-Minimal.iso` image connected as an IDE cd-rom as the installation disk, and a FTP-provided kickstart file for unattended installation.

![[Pasted image 20240517163016.png]]

Interesting, it isn't? ZFS was the absolute leader, with the various logical volumes configurations somewhat left behind. I attribute this great show to ZFS Intent Log (ZIL), but I can be wrong. At the third place we see a group including XFS, EXT4 and Qcow2 native images. At the very last place we see BTRFS, which remains slow even when disabling its CoW behavior.

If you think that installation time is a relatively unimportant operation and that BTRFS may fare well in the real benchmark, well, go ahead...

### Database prepare time

Let's see how fast (or slow) can be to prepare a PostgreSQL database via sysbench prepare command. Please note that sysbench prepare is much heavier than a “simple” raw SQL import, as it issues many synchronized writes (fsync) rather than a single sync command at the end.

![[Pasted image 20240517163211.png]]

The fastest performer was LVM with preallocated space, with preallocated Qcow2 slightly behind. ZFS is again very fast, with ThinLVM+nozeroing at its wheels.

But the real winner is the ThinLVM+EXT4 combo: it had performance very similar to a preallocated LVM volume, with the added flexibility of being a thin volume. And while you can argue that using a direct-attached ThinLVM volume without zeroing is a security hazard (at least in untrusted environments), it is relatively safe to use it as the backing device of a host-side filesystem installation.

What about the large XFS vs EXT4 gap? It can probably related to more efficient journal commit, but maybe it is another problem entirely: I noticed that with default 512KB chunks, XFS log buffer size is (by default) not optimally sized. Technical explanation: XFS log buffer size is MAX(32768, log_sunit). At filesystem creation mkfs.xfs try to optimize the sunit values for the current raid setup, but a 512KB stripe value is too much and so mkfs.xfs reverts to a sunit value of only 32KB. In fact, during filesystem creation you can see something similar:

```
log stripe unit (524288 bytes) is too large (maximum is 256KiB)
log stripe unit adjusted to 32KiB
```

This, in turn, cause a low default log buffer size. Please note that while logbsize is a tunable mount parameters (and we can indeed tune even the mkfs options), for this comparison I decided to stick with default options. So I left XFS with default settings. In a future article I'll dissect this (and others) behavior, but this is a work for another time...

Another possible reason is XFS log placement, directly at the middle of the volume: with a mostly-empty filesystem (as in this case: of our 800 GB volume, only a maximum of 32 GB where allocated) it is not the best placement. On the other side, when usage grows, it surely become a good choice.

What about BTRFS? It surprises me: it is fast when using CoW + preallocation or NoCoW + thin, and slow in the other cases. The first result puzzles me: for a CoW filesystem, preallocation don't tell very much, as new blocks will never rewritten in place.

On the other side, the fast NoCoW + thin result can be explained, as thin provision give the filesytem a chance to turn sparse writes into contiguous ones. But hey – the DB inserts performed by sysbench should already be contiguous, right? True, but don't forget the XFS layer inside the guest image: as explained, by default XFS store its journal at the middle of the volume. Thin provision means than the host can remap the log to another location (the beginning of the volume, as guest filesystem creation is one of the first thing the installer accomplishes), and so total seek time can go down (even with small 8 GB images, guest filesystem was under 20% full, so the middle-of-disk placement was not optimal). In other words, we are probably observing an interesting interaction between the host and guest filesystems, and their respective methods to allocate data/metadata blocks.

### Sysbench complex test

Ok, preparing the DB resulted in mixed results, but what about running the sysbench complex test against it?

![[Pasted image 20240517163519.png]]

At first, results are very low for all the contenders. However, you should remember that this benchmark runs concurrently to the others (fileserver, varmail and webserver), so the host had a very hard time keeping pace with the outstanding I/O operations.

Basically, any setup apart BTRFS and ZFS give similar performance. Even ZFS can be deemed acceptable. However, BTRFS is a complete fiasco here: it is 10 times slower than ZFS.

Now, have a look at latency:

![[Pasted image 20240517163541.png]]

A very similar pattern emerge. In other words, don't use BTRFS for a PostgreSQL installation, nor for any storage subsystem backing a PostgreSQL VM.

### Filebench fileserver personality 

Will the fileserver benchmark give us a different picture?

![[Pasted image 20240517163615.png]]

Mmm... not really. All the tested configurations, except BTRFS, perform reasonably well.

Latency paints a very similar picture:

![[Pasted image 20240517163625.png]]

### Filebench varmail personality 

Mail servers are an integral part of every businesses, and a busy mailserver surely impose a noticeable load on the I/O subsystem. In a similar manner to a DB server, the main problem here is the continuous stream of synchronized writes coming from the SMTP server (BBU RAID card and SSD exists for that precise reason).

![[Pasted image 20240517163644.png]]

ZFS loses some ground here, as thin XFS, but it is nothing compared to BTRFS... the benchmark reports 0 (zero) MB/s. You can argue that the benchmark was flawed, and maybe that it crashed, but the latency measurement shows us something different:

*(Image was lost.)*

BTRFS was so slow (over 10 sec. latency) that its throughput inevitable fell below the 0.0 MB/s mark (maybe it was 0.04 MB/s or the likes). Basically it was 20 times slower than the others.

### Filebench webserver personality 

Finally, a read only test...

![[Pasted image 20240517163755.png]]

Here BTRFS performs quite well, apart the CoW+prellocation combo. This was expected: as noted above, preallocation means very little to a CoW filesystem (it can even degrade performance).

Now, latency:

*(Image was lost.)*

The same apply here.

### Some considerations

First, don't get me wrong: this is not a trial against BTRFS. I would like to see it as the best performing filesystem, as it has a ton of promising features. But hey – face the reality: its performances here were really, really low. You can argue that I used a too old kernel, or that enterprise hardware has BBU RAID cards and that it should perform much better with SSDs.

Yes, yes and yes. But older kernel are a fact of life: enterprise distributions (as RedHat, CentOS and SuSE) don't ship with bleeding-edge kernels. Moreover, spinning disks are here to stay. Finally, BTRFS itself don't like HW RAID cards so much, as they interfere with retrieving the correct data in case of bitrotting.

So, while I plan to do some test on a SSD equipped system and on a old, but enterprise-grade server, BTRFS really had to perform reasonably well in the common, 7200 RPM disks case.

In the end, while BTRFS is very well suited to manage many small, rarely-changing files (eg: fileserver, NAS), it don't bode well with large, rewriting files (as VM images and databases).

BTRFS apart, what else can we tell from this benchmark session?

**Classical LVMs (preallocated volumes) remain the safer bet, performance wise.** After all, they present to the guest system a mostly contiguous disk space, reducing fragmentation. On the other side, they are not very flexible: you not only have no thin provisioning options, but dealing with raw volumes is always a little clunky (eg: for backup purpose). Moreover, LVM snapshot support is quite slow. On the other hand, LVM allow for snapshotting a single volume/VM image, mitigating the snapshot performance hit. So, if you are all for the fastest build, go with normal LVM volumes.

**Thin LVM are a good compromise:** their default coarse chunk size (64KB to 8MB, but often in the 512KB+ range) means a less fragmented allocation than CoW filesystems, and you have the added benefit of thin storage. Moreover, thin snapshot implementation is way faster than the legacy one.

**An even more interesting setup is the ThinLVM + nozeroing + filesystem combo.** You take all the advantage of thin volumes (with no zeroing-imposed speed degradation) with filesystem's typical easy of use. The fastest upper filesystem to use seem to be EXT4, but even XFS remains a very good choice.

**ZFS was a pleasant surprise:** while it is a proven, fast filesystem under Solaris / FreeBSD, its native Linux implementation is very good (and fast). It features a load of advanced characteristics, even more that BTRFS (eg: working RAID5/6 equivalent, on-line deduplication, etc). The only complaint is that, while it is a native Linux kernel module, it is not a 100% direct porting: it is based on the Sun Porting Layer, a port providing emulation of many Solaris specific APIs. This, in turn, means that it is not possibile to include ZFS support directly into mainline kernel. Anyway, if you plan to use ZFS, remember to enable xattr=sa and to use the latest RPM/DEB version provided by the project, as older version had some rare, but nasty, bugs.

## Conclusions

Well, after so many pages, what lessons can we learn?

**If you need maximum, stable performance**, even in the face of a lesser flexibility and ease to use, go with classical LVM volumes. My performance builds where based on this LVM scheme and I am very pleased of how they runs. However, do not forget to make a judicious use of snapshots, as their legacy implementation is quite slow.

**If you are ready to use a somewhat newer and more flexible (albeit less tested) technology, ThinLVM is your prime candidate**, especially when coupled with a host-side filesystem: with this later arrangement you can disable zeroing without too much security concerns. Snapshots are quite fast, also. Only remember that fragmentation can somewhat slow-down you system over time.

A similar consideration can be done for **ZFS on Linux, which shows very good results**. You will definitely like its advanced features, most notably compression (with LZ4 compressor). But if you are thinking to enable on-line deduplication, please think twice: due to how it is implemented, it both require enormous amount of RAM and often give you low space saving (but this is good for another article...)

**For VMs storage, stay well away from BTRFS:** not only it is marked a “Tech Preview” from RedHat (read: not 100% production ready), but it is very slow when used as a VM images store.

I hope you found this article interesting. If you want, you can email me at [info@assyoma.it](https://web.archive.org/web/20230330145140/mailto:info@assyoma.it?subject=ZFS, BTRFS, XFS, EXT4 and LVM with KVM – a storage performance comparison).

Have a nice day!