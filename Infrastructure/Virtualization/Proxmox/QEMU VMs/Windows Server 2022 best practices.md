---
created: 2024-02-22T14:05
updated: 2024-02-22T14:35
tags:
  - Proxmox
  - Windows
  - LVM
---
https://pve.proxmox.com/wiki/Windows_2022_guest_best_practices
### Prepare

To obtain a good level of performance, we will install the [Windows VirtIO Drivers](https://pve.proxmox.com/wiki/Windows_VirtIO_Drivers "Windows VirtIO Drivers") during the Windows installation.

- Create a new VM. 
- Select "Microsoft Windows 11/2022" as Guest OS. 
- Enable the "Qemu Agent" in the System tab. 
- Continue and mount your Windows Server 2022 ISO in the CDROM drive.
- For your virtual hard disk select "SCSI" as bus with "VirtIO SCSI" as controller. 
- Set "Write back" as cache option for best performance (the "No cache" default is safer, but slower). 
- Tick "Discard" to optimally use disk space (TRIM).
- Configure your memory settings as needed, continue and set "VirtIO (paravirtualized)" as network device, finish your VM creation.

- For the VirtIO drivers, upload the driver ISO (use the stable VirtIO ISO, [download it from here](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso)) to your storage.
- Create a new CDROM drive (use "Add -> CD/DVD drive" in the hardware tab) **with Bus "IDE" and number 0**. 
- Load the Virtio Drivers ISO in the new virtual CDROM drive.

- Now you're ready to start the VM, just follow the Windows installer.

### Launch Windows install

- Follow the installer steps until you reach the installation type selection where you need to select "Custom (advanced)"

- Now click "Load driver" to install the VirtIO drivers for hard disk and the network.
    - Hard disk: 
	    - Browse to the CD drive where you mounted the VirtIO driver and select folder "vioscsi\2k19\amd64" and confirm. 
	    - Select the "Red Hat VirtIO SCSI pass-through controller" and click next to install it. 
	    - Now you should see your drive.
    - Network: 
	    - Repeat the steps from above (click again "Load driver", etc.).
	    - Select the folder "NetKVM\2k19\amd64" and confirm it.  
	    - Select "Redhat VirtIO Ethernet Adapter" and click next.
    - Memory Ballooning: 
	    - This time select the "Balloon\2k19\amd64" folder. 
	    - Then the "VirtIO Balloon Driver" 
	    - Install it by clicking next. 
- Choose the drive and continue the Windows installer steps.

With these three drivers you should be good covered to run a fast virtualized Windows Server 2022 system.

## Install Guest Agent and Services

### Guest Agent

If you enabled the Qemu Agent option for the VM the mouse pointer will probably be off after the first boot.

To remedy this install the "Qemu Guest Agent". The installer is located on the driver CD under guest-agent\qemu-ga-x86_64.msi.

### Drivers and Services

The easiest way to install missing drivers and services is to use the provided MSI installer. It is available on the driver CD since version "[virtio-win-0.1.173-2](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/)".

Run the "virtio-win-gt-x64.msi" file located directly on the CD. If you do not plan to use [SPICE](https://pve.proxmox.com/wiki/SPICE "SPICE") you can deselect the "Qxl" and "Spice" features. Restart the VM after the installer is done.

After all this the RAM usage and IP configuration should be shown correctly in the summary page of the VM.