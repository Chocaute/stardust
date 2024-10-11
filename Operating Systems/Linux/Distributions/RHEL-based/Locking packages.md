---
created: 2024-03-12T16:27
updated: 2024-03-12T16:27
tags:
  - RHEL
  - Linux
---
#### Empêche qu'un paquet soit mis à jour ou supprimé par yum. 

Utile pour garder une version spécifique d’un paquet ou éviter des conflits de dépendances. 

Pour verrouiller un paquet, utilise le plugin yum-versionlock1. Par exemple, pour verrouiller le paquet httpd :

```shell
yum install yum-plugin-versionlock
```

```shell
yum versionlock httpd
```

Pour déverrouiller un paquet, utilise la commande :

```shell
yum versionlock delete
```