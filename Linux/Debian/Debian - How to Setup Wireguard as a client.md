---
created: 2024-03-13T17:37
updated: 2024-03-13T17:41
tags:
  - Debian
  - Wireguard
related link: https://wireguard.how/client/debian/
---
## Configure the Client
### Install WireGuard

Install the WireGuard packages. After this step, `man wg` and `man wg-quick` will work and the `wg` command gets bash completion.

```shell
apt install wireguard -y
```
```shell
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following additional packages will be installed:
  dkms linux-compiler-gcc-8-x86 linux-headers-4.19.0-16-amd64 linux-headers-4.19.0-16-common linux-headers-amd64 linux-kbuild-4.19 wireguard-dkms wireguard-tools
Suggested packages:
  python3-apport menu openresolv | resolvconf
The following NEW packages will be installed:
  dkms linux-compiler-gcc-8-x86 linux-headers-4.19.0-16-amd64 linux-headers-4.19.0-16-common linux-headers-amd64 linux-kbuild-4.19 wireguard wireguard-dkms wireguard-tools
0 upgraded, 9 newly installed, 0 to remove and 0 not upgraded.
...
DKMS: install completed.
Setting up wireguard-tools (1.0.20210223-1~bpo10+1) ...
wg-quick.target is a disabled or a static unit, not starting it.
Setting up linux-headers-4.19.0-16-common (4.19.181-1) ...
Setting up wireguard (1.0.20210223-1~bpo10+1) ...
Setting up linux-headers-4.19.0-16-amd64 (4.19.181-1) ...
Setting up linux-headers-amd64 (4.19+105+deb10u11) ...
Processing triggers for man-db (2.8.5-2) ...
```

## Get the Server Public Key

(We’re on the server for this section.)

Print the server’s public key. We’ll need this soon.

```shell
wg show wg0
```
```shell
interface: wg0
  public key: 2efuG9OYmMPQpbkJ8CVxGlvQflY6p1u+o4wjcgGII0A=
  private key: (hidden)
  listening port: 51820
```

## Back on the Client

### Create Client Keys

In every client/server relationship, each peer has its own private and public keys. Create private and public keys for the WireGuard client service. Protect the private key with a file mode creation mask.
```shell
(umask 077 && wg genkey > wg-private.key)
wg pubkey < wg-private.key > wg-public.key
```

Print the private key, we’ll need it soon.
```shell
cat wg-private.key
```
```
oBkgA+KZU6mWY5p7d0PEWxnYkihBw9TmHZXEYnQkz3g=
```

Create the WireGuard client service config file at `/etc/wireguard/wg0.conf`. (Use a command like `sudo nano /etc/wireguard/wg0.conf`.)

```shell
# define the local WireGuard interface (client)
[Interface]

# contents of file wg-private.key that was recently created
PrivateKey = oBkgA+KZU6mWY5p7d0PEWxnYkihBw9TmHZXEYnQkz3g=

# define the remote WireGuard interface (server)
[Peer]

# contents of wg-public.key on the WireGuard server
PublicKey = 2efuG9OYmMPQpbkJ8CVxGlvQflY6p1u+o4wjcgGII0A=

# the IP address of the server on the WireGuard network 
AllowedIPs = 10.0.2.1/32

# public IP address and port of the WireGuard server
Endpoint = 35.36.37.38:51820
```