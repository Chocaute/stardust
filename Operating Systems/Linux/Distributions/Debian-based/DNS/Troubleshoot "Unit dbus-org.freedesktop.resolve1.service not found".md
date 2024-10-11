---
created: 2024-03-13T16:27
updated: 2024-03-13T16:28
tags:
  - Debian
  - DNS
related link: https://superuser.com/questions/1427311/activation-via-systemd-failed-for-unit-dbus-org-freedesktop-resolve1-service
---
This problem is related to the `network-manager` and `systemd-resolved.service`.

After reading the the [manjaro forums](https://forum.manjaro.org/t/solved-systemd-resolved-resolv-journalctl-spam/81280/11) and the [arch wiki](https://wiki.archlinux.org/index.php/Systemd-resolved)

You have 2 options

## [configure network-manager to not use systemd-resolved.service](https://forum.manjaro.org/t/solved-systemd-resolved-resolv-journalctl-spam/81280/10?u=nelaaro)

```shell
vim /etc/NetworkManager/conf.d/no-systemd-resolved.conf
```

With this content:

```
[main]
systemd-resolved=false
```

## [Activate and use systemd-resolved.service](https://wiki.archlinux.org/index.php/Systemd-resolved)

check if it is running and enabled

```shell
systemctl status systemd-resolved.service
● systemd-resolved.service - Network Name Resolution
  Loaded: loaded (/usr/lib/systemd/system/systemd-resolved.service; disabled; vendor preset: enabled)
  Active: inactive (dead)
    Docs: man:systemd-resolved.service(8)
          https://www.freedesktop.org/wiki/Software/systemd/resolved
          https://www.freedesktop.org/wiki/Software/systemd/writing-network-configuration-managers
          https://www.freedesktop.org/wiki/Software/systemd/writing-resolver-clients
```

Enable and start it.

```shell
systemctl enable systemd-resolved.service
Created symlink /etc/systemd/system/dbus-org.freedesktop.resolve1.service → /usr/lib/systemd/system/systemd-resolved.service.
Created symlink /etc/systemd/system/multi-user.target.wants/systemd-resolved.service → /usr/lib/systemd/system/systemd-resolved.service.
systemctl start systemd-resolved.service
systemctl status systemd-resolved.service
● systemd-resolved.service - Network Name Resolution
  Loaded: loaded (/usr/lib/systemd/system/systemd-resolved.service; enabled; vendor preset: enabled)
  Active: active (running) since Fri 2019-04-19 10:36:53 SAST; 32min ago
    Docs: man:systemd-resolved.service(8)
          https://www.freedesktop.org/wiki/Software/systemd/resolved
          https://www.freedesktop.org/wiki/Software/systemd/writing-network-configuration-managers
          https://www.freedesktop.org/wiki/Software/systemd/writing-resolver-clients
Main PID: 21150 (systemd-resolve)
  Status: "Processing requests..."
    Tasks: 1 (limit: 4915)
  Memory: 5.0M
  CGroup: /system.slice/systemd-resolved.service
          └─21150 /usr/lib/systemd/systemd-resolved
```

That solved the errors from coming up in the logs.