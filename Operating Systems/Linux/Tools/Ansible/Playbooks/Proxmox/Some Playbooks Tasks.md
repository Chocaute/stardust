---
created: 2024-02-26T12:10
updated: 2024-03-14T23:00
tags:
  - Ansible
  - Proxmox
  - Automation
---
> [!NOTE] About
> The below requirements are needed on the host that executes this module.
> - proxmoxer (python3-proxmoxer for system-wide python package)
> - requests (python3-requests for system-wide python package)

### Create a VM
```yml
- name: Create a VM
  hosts: proxmox
  tasks:
    - name: Create the VM
      community.general.proxmox_kvm:
        node: media
        api_user: root@pam
        api_password: [REDACTED]
        api_host: media
        name: bookworm2
        vmid: 130
        ostype: 'l26'
        agent: 'true'
        cpu: 'x86-64-v2-AES'
        scsihw: 'virtio-scsi-single'
        boot: 'order=scsi0;ide2;net0'
        memory: '2048'
        ide:
          ide2: 'local-hdd:iso/debian-12.2.0-amd64-netinst.iso,media=cdrom'
        net:
          net0: 'virtio,bridge=vmbr2,firewall=1'

    - name: Create disk for VM
      community.general.proxmox_disk:
        api_user: root@pam
        api_password: [REDACTED]
        api_host: media
        vmid: 130
        disk: scsi0
        format: "raw"
        storage: local-nvme-lvm
        size: 32
        cache: "writeback"
        iothread: "true"
        discard: "on"
```

### Start a VM
```yml
- name: Start a VM
  hosts: proxmox
  tasks:
	- name: Start the VM
      community.general.proxmox_kvm:
        node: media
        api_user: root@pam
        api_password: [REDACTED]
        api_host: media
        vmid: 130
        state: started
```

### Check VM status
```yml
- name: Check VM status
  hosts: proxmox
  tasks:
	- name: Check VM status
	  community.general.proxmox_kvm:
        node: media
        api_user: root@pam
        api_password: [REDACTED]
        api_host: media
        vmid: 130
        state: current
      debug:
        msg: "VM status: {{ vm_check_result.status }}"
```

### Delete a VM
```yml
- name: Delete a Proxmox VM
  hosts: proxmox
  tasks:
    - name: Use proxmox_kvm module
      proxmox_kvm:
        api_user    : root@pam
        api_password: [REDACTED]
        api_host    : media
        name        : bookworm
        node        : media
        state       : absent
```


```yml
- name: Create a proxmox container
  hosts: proxmox
  tasks:
    - name: Create a new container
      proxmox:
        node: media
        api_user: root@pam
        api_password: [REDACTED]
        api_host: media
        hostname: OHMAGAD
        searchdomain: tomraud.fr
        nameserver: 172.16.0.1
        pubkey: 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBMJ0Tewn4nCuKogGxAWij0+QkcJMcEKprIHFgiO+GX4 root@ansibeul'
        timezone: Europe/Paris
        password: [REDACTED]
        netif: '{"net0":"name=eth0,ip=dhcp,bridge=vmbr2"}'
        disk: 'local-nvme-lvm:8'
        cores: 1
        ostemplate: 'local-hdd:vztmpl/debian-12-standard_12.2-1_amd64.tar.zst'
        features:
          - nesting=1
```

