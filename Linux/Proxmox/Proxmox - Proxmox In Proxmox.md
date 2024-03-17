---
created: 2023-12-21T17:29
updated: 2023-12-21T17:30
---
#Linux #Debian #Proxmox 

https://www.aukfood.fr/proxmox-dans-proxmox/
## Prérequis

Voici les prérequis côté serveur, donc côté serveur Proxmox hôte :

- un processeur récent supportant les instructions de virtualisation
- un kernel de version supérieur à la version 3.10, ce qui est le cas depuis longtemps sur Proxmox maintenant.
- activer le support nested

## Activation du support nested

On commence par contrôler si le paramètre nested est activé ou pas dans le processeur :

```bash
cat /sys/module/kvm_intel/parameters/nested
```

NB : par la suite, remplacer **kvm_intel** par **kvm_amd** pour les processeur AMD (logique 😉 )  
NB2 : suivant le cas la réponse est N ou 0 dans le cas du support non activé

Pour activer le support :

```bash
echo "options kvm-intel nested=1" > /etc/modprobe.d/kvm-intel.conf
```

Il faut redémarrer le module kvm. Pour cela il faut que toutes les vms soient arrêtée sur le serveur.

```bash
modprobe -r kvm-intel
modprobe kvm-intel
```

On vérifie :

```bash
# cat /sys/module/kvm_intel/parameters/nested 
```

## Côté VM

Il y a quelques obligation du côté de la vm :

- le CPU doit être du model "host"
- de la RAM
- rajouter le paramètre : **args: -enable-kvm** dans le fichier de conf de la vm (Par exemple : **/etc/pve/nodes/srv-host/qemu-server/901.conf)>/etc/pve/nodes/srv-host/qemu-server/901.conf**)

A partir de maintenant il est possible d'installer Proxmox dans une VM et dans ce Proxmox créer des VM. Bien sûr il ne faut pas s'attendre à avoir de grosses performances mais cela permet de tester des architectures.