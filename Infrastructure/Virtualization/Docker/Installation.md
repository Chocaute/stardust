---
created: 2024-05-17T18:57
updated: 2024-05-17T19:01
tags:
  - Windows
  - Linux
  - Docker
---
# Sur Linux

## Debian / Ubuntu

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

```bash
sudo docker run hello-world
```


## Windows 10

### Installation 

Téléchargez [Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows) et exécutez le programme d’installation (vous devrez vous connecter. Créez un compte si vous n’en avez pas encore). [Des instructions d’installation détaillées](https://docs.docker.com/docker-for-windows/install) sont disponibles dans la documentation de Docker.

