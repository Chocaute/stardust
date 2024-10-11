---
created: 2024-05-17T18:54
updated: 2024-05-17T19:01
tags:
  - Docker
  - Windows
  - LXC
related link: https://learn.microsoft.com/fr-fr/virtualization/windowscontainers/deploy-containers/linux-containers
---
# À propos

Dans ce modèle, le client docker s’exécute sur le bureau Windows, mais appelle le démon Docker sur la machine virtuelle Linux.

![[Pasted image 20240517190032.png]]

Dans ce modèle, tous les conteneurs Linux partagent un hôte de conteneur basé Linux. Par ailleurs, les conteneurs Linux :

- Partagent un noyau entre chacun d’eux et la machine virtuelle Moby, mais pas avec l’hôte Windows.
- Utilisent des propriétés de stockage et de mise en réseau cohérentes avec les conteneurs Linux s’exécutant sur Linux (puisqu’ils s’exécutent sur une machine virtuelle Linux).

Cela signifie également que l’hôte de conteneur Linux (machine virtuelle Moby) doit exécuter le démon Docker et toutes les dépendances de celui-ci.

Pour voir si vous opérez avec une machine virtuelle Moby, vérifiez le Gestionnaire Hyper-V pour machine virtuelle Moby soit en utilisant l’interface utilisateur du Gestionnaire Hyper-V, soit en exécutant `Get-VM` dans une fenêtre PowerShell avec élévation de privilèges.
# Configuration

-> Panneau de configuration -> Activer ou désactiver des fonctionnalités Windows.

![[Pasted image 20240423172243.png]]

Pour exécuter des conteneurs Linux, vous devez vous assurer que Docker cible le démon approprié. Vous pouvez activer cette option en sélectionnant `Switch to Linux Containers` dans le menu Action après avoir cliqué sur l’icône de la baleine Docker dans la barre d’état système. Si vous voyez `Switch to Windows Containers`, cela signifie que vous ciblez déjà le démon Linux.

![[Pasted image 20240423172342.png]]

Une fois que vous avez confirmé que vous ciblez le bon démon, exécutez le conteneur avec la commande suivante :

```bash
docker run --rm busybox echo hello_world
```

Le conteneur doit s’exécuter, afficher "hello_world", puis se fermer.

Lorsque vous interrogez `docker images`, vous devez voir l’image conteneur Linux que vous venez juste de tirer (pull) et d’exécuter :

```
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

# Créer votre environnement. 

- Le ficher est à adapter. 
- Les trois machines virtuelles existeront dans un sous-réseau local.
- `sudo docker exec -it debian1 bash` pour accèder directement à un container.

docker-compose.yml
```yaml
services:
  debian1:
    image: debian
    networks:
      mynet:
        ipv4_address: 172.20.0.2
    container_name: debian1
    command: tail -f /dev/null  # Keep container running
    restart: unless-stopped
    volumes:
      - ./debian1:/opt/

  debian2:
    image: debian
    networks:
      mynet:
        ipv4_address: 172.20.0.3
    container_name: debian2
    command: tail -f /dev/null  # Keep container running
    restart: unless-stopped
    volumes:
      - ./debian2:/opt/

  debian3:
    image: debian
    networks:
      mynet:
        ipv4_address: 172.20.0.4
    container_name: debian3
    command: tail -f /dev/null  # Keep container running
    restart: unless-stopped
    volumes:
      - ./debian3:/opt/

networks:
  mynet:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 172.20.0.0/24
```