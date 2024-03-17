---
created: 2024-03-15T13:02
updated: 2024-03-15T13:03
tags:
  - Ansible
  - Linux
  - Automation
  - AD
---
```yml
- name: Create realmd.conf file with OS information
  hosts: bookworm2
  gather_facts: yes
  become: yes
  tasks:
    - name: Create realmd.conf file
      template:
        src: templates/realmd.conf.j2
        dest: /etc/realmd.conf
      vars:
        os_name: "{{ ansible_distribution }}"
        os_version: "{{ ansible_distribution_version }}"
```
