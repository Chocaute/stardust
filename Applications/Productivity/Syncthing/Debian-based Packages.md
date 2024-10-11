---
created: 2024-04-02T09:40
updated: 2024-10-11T13:51
tags:
  - Syncthing
  - Debian
related link: https://apt.syncthing.net/
---
### Add the release PGP keys:
```shell
sudo mkdir -p /etc/apt/keyrings
sudo curl -L -o /etc/apt/keyrings/syncthing-archive-keyring.gpg https://syncthing.net/release-key.gpg
```

### Add the "stable" channel to your APT sources:
```shell
echo "deb [signed-by=/etc/apt/keyrings/syncthing-archive-keyring.gpg] https://apt.syncthing.net/ syncthing stable" | sudo tee /etc/apt/sources.list.d/syncthing.list
```

### Update and install syncthing:
```shell
sudo apt-get update
sudo apt-get install syncthing
```

### Then configure [[As a Service (systemd)]]