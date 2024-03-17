---
created: 2024-03-15T13:01
updated: 2024-03-15T13:02
tags:
  - Ansible
  - Linux
  - Automation
  - AD
---
```yml
- name: Enable mkhomedir using pam-auth-update
  hosts: bookworm2
  tasks:
    - name: Run pam-auth-update command
      become: yes
      become_user: root
      command: pam-auth-update --enable mkhomedir
```

