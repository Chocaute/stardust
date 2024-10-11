---
created: 2024-04-02T09:02
updated: 2024-04-02T09:17
tags:
  - Proxmox
  - Networking
related link: https://pve.proxmox.com/wiki/Software-Defined_Network
---
> [!INFO] The **S**oftware-**D**efined **N**etwork (SDN) feature in Proxmox VE 
> Enables the creation of virtual zones and networks (VNets). This functionality simplifies advanced networking configurations and multitenancy setup.

### Introduction
The Proxmox VE SDN allows for separation and fine-grained control of virtual guest networks, using flexible, software-controlled configurations.

Separation is managed through **zones**, virtual networks (**VNets**), and **subnets**. A zone is its own virtually separated network area. A VNet is a virtual network that belongs to a zone. A subnet is an IP range inside a VNet.

Depending on the type of the zone, the network behaves differently and offers specific features, advantages, and limitations.

Use cases for SDN range from an isolated private network on each individual node to complex overlay networks across multiple PVE clusters on different locations.

After configuring an VNet in the cluster-wide datacenter SDN administration interface, it is available as a common Linux bridge, locally on each node, to be assigned to VMs and Containers.

### Support status
> [!Quote]- History
> The Proxmox VE SDN stack has been available as an experimental feature since 2019 and has been continuously improved and tested by many developers and users. With its integration into the web interface in Proxmox VE 6.2, a significant milestone towards broader integration was achieved. During the Proxmox VE 7 release cycle, numerous improvements and features were added. Based on user feedback, it became apparent that the fundamental design choices and their implementation were quite sound and stable. Consequently, labeling it as ‘experimental’ did not do justice to the state of the SDN stack. For Proxmox VE 8, a decision was made to lay the groundwork for full integration of the SDN feature by elevating the management of networks and interfaces to a core component in the Proxmox VE access control stack. In Proxmox VE 8.1, two major milestones were achieved: firstly, DHCP integration was added to the IP address management (IPAM) feature, and secondly, the SDN integration is now installed by default.

> [!Abstract]- Current Status 02/04/2024
> The current support status for the various layers of our SDN installation is as follows:
> 
> - Core SDN, which includes VNet management and its integration with the Proxmox VE stack, is fully supported.
>     
> - IPAM, including DHCP management for virtual guests, is in tech preview.
>     
> - Complex routing via FRRouting and controller integration are in tech preview.

### Installation
#### SDN Core
Since Proxmox VE 8.1 the core Software-Defined Network (SDN) packages are installed by default.

If you upgrade from an older version, you need to install the `libpve-network-perl` package on every node:
```shell
apt update
apt install libpve-network-perl
```

> [!warning]
> Proxmox VE version 7.0 and above have the ifupdown2 package installed by default. If you originally installed your system with an older version, you need to explicitly install the ifupdown2 package.
> 

After installation, you need to ensure that the following line is present at the end of the `/etc/network/interfaces` configuration file on all nodes, so that the SDN configuration gets included and activated.

```shell
source /etc/network/interfaces.d/*
```