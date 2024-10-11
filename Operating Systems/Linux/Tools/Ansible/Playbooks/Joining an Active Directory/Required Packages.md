---
created: 2024-02-26T12:34
updated: 2024-03-15T13:08
tags:
  - Ansible
  - Linux
  - Automation
  - AD
---
```yml
- name: Install required packages
  hosts: bookworm2
  become: yes
  tasks:
    - name: Install packages with apt
      apt:
        name:
          - realmd
          - sssd
          - sssd-tools
          - libnss-sss
          - libsss-sudo
          - libpam-sss
          - adcli
          - samba-common-bin
          - oddjob
          - oddjob-mkhomedir
          - packagekit
        state: present
```