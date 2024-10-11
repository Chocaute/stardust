---
created: 2024-03-18T15:22
updated: 2024-03-20T17:09
---
Documentation interne

From LXC to docker container

  

- Minimum prerequisites :
    

- Access to the rootfs folder of the LXC of interest.
    
- Docker somewhere. 
    

  

## In a nutshell

On Proxmox, mount the CT volume:

  

|   |
|---|
|pct mount {id}|

  

Import the rootfs folder as a docker image:

  

|   |
|---|
|tar -C /var/lib/lxc/{id}/rootfs -c . \| docker import - {imagename:tagname}|

  

Create a container from the image:

  

|   |
|---|
|docker run -d --name {containername} -p 8000:80 {imagename:tagname} tail -f /dev/null|

  

## With Proxmox and Docker on two different hosts 

### In a shell

On the Proxmox host, mount the LXC and extract its rootfs:

  

|   |
|---|
|pct mount {id}<br><br>cp -rp /var/lib/lxc/{id}/rootfs /tmp/rootfs<br><br>pct unmount {id}|

  

Archive the copied rootfs folder then delete it:

  

|   |
|---|
|tar -czvf rootfs.tar.gz /tmp/rootfs<br><br>rm -r /tmp/rootfs|

  

Scp it to the docker server:

  

|   |
|---|
|scp rootfs.tar.gz {docker-user}@{docker-server}:/home/{docker-user}/<br><br>rm /tmp/rootfs.tar.gz|

  

On the Docker host, untar the archive:

  

|   |
|---|
|tar -xzf rootfs.tar.gz<br><br>rm rootfs.tar.gz|

  

Then import the rootfs folder as a docker image:

  

|   |
|---|
|tar -C ./rootfs -c . \| sudo docker import - {imagename:tagname}<br><br>rm -r rootfs|

  

You can now create a container from the image:

  

|   |
|---|
|docker run -d --name {containername} -p 8000:80 {imagename:tagname} tail -f /dev/null|

  
  
  

### In an Ansible playbook 

Don’t forget to adapt this playbook to your needs.

  

|   |
|---|
|- name: Transfer rootfs from Proxmox to Docker server<br><br>  hosts: proxmox<br><br>  tasks:<br><br>- name: Mount container volume<br><br>   shell: pct mount {id}<br><br>  <br><br>- name: Copy rootfs to temporary directory<br><br>   shell: cp -rp /var/lib/lxc/{id}/rootfs /tmp/rootfs<br><br>  <br><br>- name: Unmount container volume<br><br>   shell: pct unmount {id}<br><br>  <br><br>- name: Create tar.gz archive<br><br>   shell: tar -czvf rootfs.tar.gz -C /tmp/ rootfs/<br><br>   args:<br><br>     chdir: /tmp<br><br>  <br><br>- name: Transfer archive to Docker server<br><br>   shell: scp /tmp/rootfs.tar.gz {docker-user}@{docker-server}:/home/{docker-user}/<br><br>  <br><br>- name: Remove temporary files<br><br>   file:<br><br>     path: /tmp/rootfs.tar.gz<br><br>     state: absent<br><br>   register: remove_tmp_files<br><br>  <br><br>- name: Import rootfs as Docker image and run container<br><br>  hosts: docker_server<br><br>  tasks:<br><br>- name: Untar archive<br><br>   unarchive:<br><br>     src: /home/{docker-user}/rootfs.tar.gz<br><br>     dest: /home/{docker-user}<br><br>     remote_src: yes<br><br>  <br><br>- name: Import rootfs folder as Docker image<br><br>   shell: tar -C /home/{docker-user}/rootfs -c . \| sudo docker import - {imagename:tagname}<br><br>  <br><br>- name: Delete extracted archive<br><br>   shell: rm -r /home/{docker-user}/rootfs|

## Updating your image from the docker container

Say you made changes from within your docker container, and would like to push those changes to your local image. 

First, commit the changes you made to the container to a new image.

  

|   |
|---|
|docker commit {containername}  {new_imagename}|

  

Then tag the new image with the same name as the existing image so that it replaces the old one.

  

|   |
|---|
|docker tag {new_imagename} {imagename:tagname}:latest|

  

Optionally, you can remove the old image to avoid confusion. 

  

|   |
|---|
|docker rmi {imagename:tagname}|

  

Note: Watchtower can do this automatically and more gracefully, but I haven’t looked into it yet.

```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp

auto vmbr0
iface vmbr0 inet static
        address 10.10.10.1/24
        bridge-ports none
        bridge-stp off
        bridge-fd 0

        post-up   echo 1 > /proc/sys/net/ipv4/ip_forward
        post-up   iptables -t nat -A POSTROUTING -s '10.10.10.0/24' -o eno1 -j MASQUERADE
        post-down iptables -t nat -D POSTROUTING -s '10.10.10.0/24' -o eno1 -j MASQUERADE
        post-up   iptables -t raw -I PREROUTING -i fwbr+ -j CT --zone 1
        post-down iptables -t raw -D PREROUTING -i fwbr+ -j CT --zone 1

        post-up iptables -t nat -A PREROUTING -p tcp --dport 22 -i eno1 -d 192.168.3.111 -m conntrack --ctstate NEW -j DNAT --to 10.10.10.11:22
        post-down iptables -t nat -D PREROUTING -p tcp --dport 22 -i eno1 -d 192.168.3.111 -m conntrack --ctstate NEW -j DNAT --to 10.10.10.11:22

source /etc/network/interfaces.d/*
```

```
root@tools:/tmp# ip route
default via 192.168.2.1 dev eno1 
10.10.10.0/24 dev vmbr0 proto kernel scope link src 10.10.10.1 
192.168.2.0/23 dev eno1 proto kernel scope link src 192.168.3.111
```

____

d'abord tu me rassemble les points abordés dans une doc comme je t'ai demandé stp

ensuite tu me mets les étapes de ton plan d'action pour faire tes tests avec LXC

![[Pasted image 20240319171924.png]]

Env sur windows -> non

Tech de container -> LXC (plus simple à comprendre que Docker)

Réseau interne et local -> Bridges Linux (bridge-utils) et iptables. 

lxc vs docker, les plus et les moins
___

Prend ce /etc/network/interfaces

```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp

auto vmbr0
iface vmbr0 inet static
        address 10.10.10.1/24
        bridge-ports none
        bridge-stp off
        bridge-fd 0

        post-up   echo 1 > /proc/sys/net/ipv4/ip_forward
        post-up   iptables -t nat -A POSTROUTING -s '10.10.10.0/24' -o eno1 -j MASQUERADE
        post-down iptables -t nat -D POSTROUTING -s '10.10.10.0/24' -o eno1 -j MASQUERADE
        post-up   iptables -t raw -I PREROUTING -i fwbr+ -j CT --zone 1
        post-down iptables -t raw -D PREROUTING -i fwbr+ -j CT --zone 1

source /etc/network/interfaces.d/*
```

- L'interface physique "eth0" est en dhcp, c'est l'interface qui fait face au reste du monde.

- Vmbr0 est un bridge linux. C'est une interface virtuel qui fera office de gateway pour le sous-réseau interne et virtuel "10.10.10.0/24".

bridge-ports none

Signifie que le bridge ne gère pas directement une interface physique.

post-up   echo 1 > /proc/sys/net/ipv4/ip_forward

Active le routage.

post-up   iptables -t nat -A POSTROUTING -s '10.10.10.0/24' -o eno1 -j MASQUERADE

NAT et rend anonyme les communications sortantes du LAN de vmbr0.

post-up   iptables -t raw -I PREROUTING -i fwbr+ -j CT --zone 1

Ceci est du debug pour proxmox. Je ne sais pas si ça sera nécessaire pour un desktop, faut tester. En soit pas obligatoire.Et pouf, t'a un LAN dans ton PC. Le bridge sera utilisé pour les interfaces virtuelles au sein de l'hôte lui-même. En gros, pour les VMs et les LXC. (modifié) 

[16 h 54](https://lundimatingroupesiege.slack.com/archives/D06M9AJA4TS/p1710863649723529)

La table de routage ressemblera à quelque chose comme ça :  

root@tools:/tmp# ip route
default via 192.168.2.1 dev eth0 
10.10.10.0/24 dev vmbr0 proto kernel scope link src 10.10.10.1 
192.168.2.0/23 dev eth0 proto kernel scope link src 192.168.3.111


___

auto-creation of popular services

auto-creation of html pages into dooku-wiki (say, for backup jobs)

template for docs? 

Find something markdown. Can dookuwiki do markdown?

___

- Les commandes pour créer un LXC.
Ok.

- Les commandes pour gérer un LXC.
Ok.
IP ?

- Les configurations requises pour exécuter les LXC en nesting et unpriviliged. 
Ok pour unpriviliged.
Nesting ?

- Les commandes pour backup un LXC.
Snapshot existe.

___
```shell
cat /etc/pve/nodes/tools/lxc/107.conf 
```

```yml
arch: amd64
cores: 1
features: nesting=1
hostname: glpi
memory: 512
net0: name=eth0,bridge=vmbr0,firewall=1,gw=10.10.10.1,hwaddr=BC:24:11:9B:B5:0A,ip=10.10.10.12/24,type=veth
onboot: 1
ostype: debian
parent: stable
rootfs: local-lvm:vm-107-disk-0,size=8G
swap: 512
tags: prod
unprivileged: 1
```

Ok donc, comment transcrire ça avec LXC tout court ?

Des fichiers confs individuels existent. Mais la syntaxe est différente. 

___

think to reinstall busybox on that machine!

___

```
[connection]
id=vmbr0
uuid=12345678-1234-1234-1234-1234567890ab
type=bridge
autoconnect-priority=-999

[bridge]
interface-name=vmbr0
stp=false
forward-delay=0

[ipv4]
method=manual
address1=10.10.10.1/24
gateway=0.0.0.0
dns=0.0.0.0;

[ipv6]
method=ignore

[proxy]
```



        post-up   echo 1 > /proc/sys/net/ipv4/ip_forward
        post-up   iptables -t nat -A POSTROUTING -s '10.10.10.0/24' -o eno1 -j MASQUERADE
        post-down iptables -t nat -D POSTROUTING -s '10.10.10.0/24' -o eno1 -j MASQUERADE
        post-up   iptables -t raw -I PREROUTING -i fwbr+ -j CT --zone 1
        post-down iptables -t raw -D PREROUTING -i fwbr+ -j CT --zone 1
