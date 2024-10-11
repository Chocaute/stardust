---
created: 2024-03-12T16:41
updated: 2024-03-15T14:03
tags:
  - Proxmox
  - LXC
  - Linux
related link: https://kcore.org/2022/02/05/lxc-subuid-subgid/
---
##### Configuring the host to allow mapping

On the host OS, there are two files which play a major role: `/etc/subuid` and `/etc/subgid`. These contain the mappings of the UID’s that can be remapped.

By default they contain:

```shell
root:100000:65536
```

which means that the user/group root can map from UID/GID `100000`, and for `65536` consequitive ID’s.

Keeping in mind that LXC is started as `root` on Proxmox, this will mean that inside LXC a process started with UID 0 will be remapped to UID 100000 on the host, UID 1 will be 100001, UID 65536 will be 165536. Same with groups.

Now, if we want to allow a container to map to a UID/GID on the host, we’ll have to specifically allow it. Say, you want to actually use UID 1002, and make it match the host UID. In that case, we add this to `/etc/subuid`:

```shell
root:1002:1
```

This means that user root can map UID 1002, and it can do that for 1 sequential UID. So _only_ 1002.

Now, say that you want to map UID 1002-1005.

```shell
root:1002:4
```

Read: root can map from 1002 onwards, and for a max of 4 UID’s. This is 1002, 1003, 1004 and 1005.

##### Configuring the container[Permalink](https://kcore.org/2022/02/05/lxc-subuid-subgid/#configuring-the-container "Permalink")

In `/etc/lxc/pve/XXX.conf`, you need to add a line telling LXC to map the UID/GID. For example, to map UID 1002:

```shell
lxc.idmap: u 0    100000 1002
lxc.idmap: u 1002 1002   1
lxc.idmap: u 1003 101003 64533
```

Let’s unwrap this:

> `u` means user id mapping. You also have `g` for group id mapping.  
> `0` is the start UID to start mapping.  
> `100000` is the UID to map to  
> `1002` is the amount of UID’s to remap.

UID 0 to 1001 is mapped on UID 100000 to 101001.

> `1002` is the start UID to start mapping.  
> `1002` is the UID to map to  
> `1` is the amount of UID’s to remap.

UID 1002 is mapped to 1002, and for 1 UIDs.

> `1003` is the start UID to start mapping.  
> `101003` is the UID to map to  
> `64533` is the amount of UID’s to remap.

UID 1003 to 65536 is mapped to 101003 and on, to 165536.

This now matches with what’s in `/etc/subuid`, and should work.

A restart of the container later the UID’s should be mapped correctly.

___
##### A discussion about it on Reddit
https://old.reddit.com/r/france/comments/11lpz22/mercredi_tech_20230308/jbfbqew/?context=3

Bonjour,

Il y aurai t'il quelques personnes qui s'y connaisse en proxmox, et plus particulièrement en container LXC ?

J'pense pas que je trouverai réponse ici, mais hey, worth a shot.

En gros, j'essaie de map les groupes 1000 et 103 en même temps sur un container LXC.

Le groupe 1000, c'est pour les bind mounts. Dans le fichier de configuration du container:

```
lxc.idmap: u 0 100000 1000
lxc.idmap: g 0 100000 1000
lxc.idmap: u 1000 1000 1
lxc.idmap: g 1000 1000 1
lxc.idmap: u 1001 101001 64535
lxc.idmap: g 1001 101001 64535
```

Le groupe 103, c'est pour le relier à son homologue 105 dans le container. Dans la conf:

```
lxc.idmap: u 0 100000 65535
lxc.idmap: g 0 100000 105
lxc.idmap: g 105 103 1
lxc.idmap: g 106 100106 65430
```

Individuellement, ils fonctionnent tout les deux. Mais quand j’essaie de les faire fonctionner ensemble, j’obtiens une erreur et le container ne se lance pas.

Sachant qu’intrinsèquement je comprend rien à tout ce bazars ET que c'est un problème très spécifique, Google n'aide pas et je suis complètement québlo.

---

[–][Belenoi](https://old.reddit.com/user/Belenoi)   Suède 2 points 1 month ago* 

A priori, il faut que tu le fasses dans l'ordre, sans écraser tes changements:

```
lxc.idmap: u 0 100000 1000
lxc.idmap: u 1000 1000 1
lxc.idmap: u 1001 101001 64535
lxc.idmap: g 0 100000 105
lxc.idmap: g 105 103 1
lxc.idmap: g 106 100106 894
lxc.idmap: g 1000 1000 1
lxc.idmap: g 1001 101001 64535
```

---

[–][Chokawai](https://old.reddit.com/user/Chokawai) 1 point 1 month ago* 

Yup, ça fonctionne. Merci beaucoup, j'vais lire l'article de ce pas.

Juste une chose - j'ai demandé sur [r/proxmox](https://old.reddit.com/r/proxmox) juste avant, et un utilisateur m'a donné une solution identique, à l'exception d'une chose - 894 était de son côté 897.

Quelle était l'erreur, exactement ? Et disons que je souhaite ajouter un autre groupe, je fait comment ?

EDIT: Même après avoir lus le lien, non , j'comprend toujours pas. Pourquoi "lxc.idmap: g 0 100000 1000" doit disparaitre, par exemple? Tout ceci est terriblement confus.

RE-EDIT: C'est encore un peu vague dans ma tête, mais j'commence actuellement à biter le truc. J'ai réussi à caser un troisième groupe.

---

[–][Belenoi](https://old.reddit.com/user/Belenoi)   Suède 1 point 1 month ago 

```
lxc.idmap: u A B C
```

A : uid du container B: uid de l’hôte C: Plage a assigner

Avec

```
lxc.idmap: u 0 100000 1000
```

Tu map les uid 0->999 du container aux uid 100000->100999 de l'hôte. Le problème, c'est qu'en ajoutant d'autres lignes, tu risques d’écraser ce mapping.

Dans ton cas, j'imagine que la ligne:

```
lxc.idmap: g 106 100106 65430 
```

écrasait le changement

```
lxc.idmap: g 1000 1000 1
```

Puisque tu as associé 2 fois l'adresse 1000 du container a deux adresses différentes (1000 et 101000 respectivement).

___

```
lxc.cgroup2.devices.allow: c 226:0 rwm
lxc.cgroup2.devices.allow: c 226:128 rwm
lxc.mount.entry: /dev/dri/card0 dev/dri/card0 none bind,optional,create=file
lxc.mount.entry: /dev/dri/renderD128 dev/dri/renderD128 none bind,optional,create=file
lxc.idmap: u 0 100000 1000
lxc.idmap: u 1000 1000 1
lxc.idmap: u 1001 101001 64535
lxc.idmap: g 0 100000 44
lxc.idmap: g 44 44 1
lxc.idmap: g 45 100045 60
lxc.idmap: g 105 103 1
lxc.idmap: g 106 100106 894
lxc.idmap: g 1000 1000 1
lxc.idmap: g 1001 101001 64535
```

