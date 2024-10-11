---
created: 2024-03-13T17:14
updated: 2024-03-13T17:16
tags:
  - Linux
  - VPN
  - Wireguard
related link: https://github.com/angristan/wireguard-install
---
> WireGuard is a point-to-point VPN that can be used in different ways. Here, we mean a VPN as in: the client will forward all its traffic through an encrypted tunnel to the server. The server will apply NAT to the client's traffic so it will appear as if the client is browsing the web with the server's IP.

- The script supports both IPv4 and IPv6.
- WireGuard does not fit your environment? Check out [openvpn-install](https://github.com/angristan/openvpn-install).
## Requirements

Supported distributions:

-   AlmaLinux >= 8
-   Arch Linux
-   CentOS Stream >= 8
-   Debian >= 10
-   Fedora >= 32
-   Oracle Linux
-   Rocky Linux >= 8
-   Ubuntu >= 18.04
## Usage

Download and execute the script. Answer the questions asked by the script and it will take care of the rest.

```shell
curl -O https://raw.githubusercontent.com/angristan/wireguard-install/master/wireguard-install.sh
chmod +x wireguard-install.sh
./wireguard-install.sh
```

It will install WireGuard (kernel module and tools) on the server, configure it, create a systemd service and a client configuration file.

Run the script again to add or remove clients!