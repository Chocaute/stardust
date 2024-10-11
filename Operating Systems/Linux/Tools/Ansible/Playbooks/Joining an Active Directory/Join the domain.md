---
created: 2024-03-15T13:03
updated: 2024-03-15T13:04
tags:
  - Ansible
  - Automation
  - Linux
  - AD
---
```yml
- name: Join the realm with password
  hosts: bookworm2
  become: yes
  tasks:
    - name: Join the realm using realm join command
      expect:
        command: realm join tomraud.fr --user=administrateur
        responses:
          "Password for administrateur:": [REDACTED]
```