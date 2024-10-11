---
created: 2024-03-12T16:26
updated: 2024-03-12T16:26
tags:
  - RHEL
  - Linux
---
#### Ces commandes n'affectent pas les fichiers de configuration

Revenir à l’état précédent d’un paquet ou du système après une mise à jour ou une installation avec yum :

```shell
yum history list
```

```shell
yum history undo
```

```shell
yum history redo
```

Rétrograder un paquet à la version précédente ou à une version spécifique :

```shell
yum downgrade 
```

Spécifier une version mineure de RHEL et utiliser yum pour mettre à jour ou rétrograder les paquets :

```shell
subscription-manager release --set
```