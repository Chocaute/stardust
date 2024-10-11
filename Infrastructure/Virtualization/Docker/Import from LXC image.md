---
created: 2024-02-26T14:47
updated: 2024-02-27T16:03
tags:
  - LM
  - Docker
  - Containers
related link: https://www.buzzwrd.me/index.php/2021/03/10/creating-lxc-containers-from-docker-and-oci-images/
---
### 1. Get an LXC image: 
http://download.proxmox.com/images/system/

For my use case, I downloaded Debian 11. 

### 2. Extract the tar.zst archive
```shell
mkdir archive
tar --use-compress-program=unzstd -xvf archive-filename.tar.zst -C archive
```

### 3. Create the Docker image
```shell
tar -C ./archive -c . | sudo docker import - project/image
```

### 4. Create a container from the image
```shell
docker run -it project/image /bin/bash
```

___

mount the container volume
```bash
pct mount 115
```

cp to /tmp (archiving doesn't work well in the source folder, for some reason)
```bash
cp -rp /var/lib/lxc/115/rootfs /tmp/rootfs 
pct unmount 115
```

tar.gz the folder
```bash
tar -czvf rootfs.tar.gz /tmp/rootfs 
rm -r /tmp/rootfs
```

scp it to docker server
```bash
scp /tmp/rootfs.tar.gz tomm@10.10.10.5:/home/tomm 
rm /tmp/rootfs.tar.gz
```

untar the archive
```bash
tar -xzf rootfs.tar.gz 
rm rootfs.tar.gz
```

import the rootfs folder as a docker image
```bash
sudo tar -C ./rootfs -c . | sudo docker import - debian/infralm
sudo rm -r rootfs
```

provide a docker-compose.yml
```yml
version: '3'

services:
  debian:
    image: debian/infralm
    container_name: infralm
    command: tail -f /dev/null
    ports:
      - "8000:80"
      - "2000:20"
      - "4430:443"
```

create a container with that docker-compose
```cmd
sudo docker-compose up -d
```

> [!NOTE] 
> The `command: tail -f /dev/null` is often used as a workaround to keep the container running indefinitely.
> 
> When you start a container without specifying a command, Docker will execute the default command defined in the Dockerfile. In some cases, this default command might not keep the container running, leading Docker to exit the container immediately after starting it. By specifying `tail -f /dev/null` as the command, you ensure that the container stays running indefinitely, allowing you to interact with it or attach to its shell.

Enter this container
```cmd
docker exec -it infralm bash
```

Ansible playbook:
```yml
---
- name: Transfer rootfs from Proxmox to Docker server
  hosts: proxmox
  become: true
  tasks:
    - name: Mount container volume
      shell: pct mount 115

    - name: Copy rootfs to temporary directory
      shell: cp -rp /var/lib/lxc/115/rootfs /tmp/rootfs

    - name: Unmount container volume
      shell: pct unmount 115

    - name: Create tar.gz archive
      shell: tar -czvf rootfs.tar.gz -C /tmp/ rootfs/
      args:
        chdir: /tmp

    - name: Transfer archive to Docker server
      shell: scp /tmp/rootfs.tar.gz tomm@10.10.10.5:/home/tomm/rootfs.tar.gz

    - name: Remove temporary files
      file:
        path: /tmp/rootfs.tar.gz
        state: absent
      register: remove_tmp_files

- name: Import rootfs as Docker image and run container
  hosts: docker_server
  become: true
  tasks:
    - name: Untar archive
      unarchive:
        src: /home/tomm/rootfs.tar.gz
        dest: /home/tomm
        remote_src: yes

    - name: Import rootfs folder as Docker image
      shell: tar -C /home/tomm/rootfs -c . | sudo docker import - debian/infralm

    - name: Delete extracted archive
      shell: rm -r /home/tomm/rootfs

    - name: Provide docker-compose.yml
      template:
        src: docker-compose.yml.j2
        dest: /home/tomm/docker-compose.yml

    - name: Create container with docker-compose
      shell: docker-compose up -d
      args:
        chdir: /home/tomm
```