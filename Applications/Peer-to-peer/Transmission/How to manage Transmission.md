---
created: 2024-03-13T17:24
updated: 2024-03-13T17:30
tags:
  - Debian
  - P2P
  - Transmission
related link: https://wiki.debian.org/Transmission
---
# About

Transmission is a light-weight and cross-platform [BitTorrent](https://wiki.debian.org/BitTorrent) client with full encryption, DHT, µTP, PEX and Magnet Link support.
# Installation

If you have installed your Debian Gnome desktop from [tasksel](https://wiki.debian.org/tasksel) then [transmission-gtk](https://packages.debian.org/transmission-gtk "DebianPkg") should be installed by default, but it is possible to use one more of the following packages, depending on your requirements:

-   [transmission-cli](https://packages.debian.org/transmission-cli "DebianPkg") - A collection of cli tools for transmission.
    
-   [transmission-gtk](https://packages.debian.org/transmission-gtk "DebianPkg") - A BitTorrent client using a GTK interface.
    
-   [transmission-qt](https://packages.debian.org/transmission-qt "DebianPkg") - A BitTorrent client using a QT interface.
## Server installation

On a server you can install a BitTorrent daemon using a web interface with apt, installing also suggested packages 'cause you might need upnp and other things.

```shell
apt install transmission-daemon  --install-suggests
```

# Configuration

The [transmission-gtk](https://packages.debian.org/transmission-gtk "DebianPkg") and [transmission-qt](https://packages.debian.org/transmission-qt "DebianPkg") clients can be configured via the application or via ~/.config/transmission/settings.json.  
The [transmission-daemon](https://packages.debian.org/transmission-daemon "DebianPkg") can be configured by editing `/etc/transmission-daemon/settings.json`.

## Configurating settings.json

Changes made manually to the file `/etc/transmission-daemon/settings.json` while the daemon is running will be silently overwritten on transmission exit. You need to stop it first, make changes and start it again.

```shell
service transmission-daemon stop
```
```shell
mcedit /etc/transmission-daemon/settings.json
```
```shell
service transmission-daemon start
```

The `setting.json` file is quite self explanatory but the common things that might need editing are:  
(Full explanation [here](https://github.com/transmission/transmission/wiki/Editing-Configuration-Files))

```json
-   "download-dir": "/var/lib/transmission-daemon/downloads": the location of the finished downloads
-   "rpc-password": "*(Hh09ajdf-9djfd89ash7a8ggG&*g98h8009hj90": the password for the web interface, replace the hash with a plain text password and it will be hashed on reload

-   "rpc-username": "transmission": user name for web interface
-   "rpc-whitelist": "127.0.0.1": IPs allowed to access the daemon, something like "rpc-whitelist": "127.0.0.1,192.168.*.*", is the formatting for the IP addresses
```  

## Change default user on the daemon

In case you want to set a different user than the default debian-transmission user, you need to create a configuration file to overwrite default settings.

```shell
systemctl edit transmission-daemon.service
```

Then add these lines:

```shell
[Service]
User=USER
```

Where "USER" is the name of the user you want to run the daemon as.

The file is then located in `/etc/systemd/system/transmission-daemon.service.d/override.conf`.  
Or you can edit directly the file `/etc/systemd/system/multi-user.target.wants/transmission-daemon.service`.

## Change permissions on download directory

If you have transmission on a server, you might want to edit downloaded files from local stations. Good choice is to make the download directory editable by a group of your stations users.

Change `umask` in `settings.json` to `umask = 0` or even `umask = 7` (decimal explanation, the json markup language only accepts numbers in base 10). Finally change umask of the transmission daemon by adding `UMask=007 to /etc/systemd/system/multi-user.target.wants/transmission-daemon.service`, which change permissions of the download directory and probably change group of transmission daemon default user.

# Connecting to the daemon

## Web Interface

Once the daemon is running, it can be accessed from your web browser by pointing it at http://127.0.0.1:9091 or if you come from local network http://ip-address-of-server:9091 (once you have whitelisted the IP range).

## Command line app CLI and TUI

For connecting to a daemon from the shell you can use stig ([https://github.com/rndusr/stig](https://github.com/rndusr/stig)). You can manipulate torrents and get stats. For example, with stig ls active you get active torrents.

## Other possibilities

The are plenty of other ways to access the daemon - see [https://transmissionbt.com/resources/](https://transmissionbt.com/resources/).

# External Links

-   [http://www.transmissionbt.com/](http://www.transmissionbt.com/)
-   [https://github.com/transmission/transmission/wiki](https://github.com/transmission/transmission/wiki) - Transmission wiki