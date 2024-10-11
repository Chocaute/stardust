---
created: 2024-03-15T13:04
updated: 2024-03-15T13:06
tags:
  - Ansible
  - Linux
  - Automation
  - AD
---
```yml
- name: Update sssd.conf
  hosts: bookworm2
  become: yes
  tasks:
    - name: Update sssd.conf 1/2
      lineinfile:
        path: /etc/sssd/sssd.conf
        regexp: '^use_fully_qualified_names\s*='
        line: 'use_fully_qualified_names = False'

    - name: Update sssd.conf 2/2
      lineinfile:
        path: /etc/sssd/sssd.conf
        regexp: '^services\s*='
        line: 'services = nss, pam, sudo'

#    - name: Add sudoers configuration to nsswitch.conf
#      blockinfile:
#        path: /etc/nsswitch.conf
#        block: |
#          sudoers: files sss

	- name: Restart sssd service
      systemd:
        name: sssd
        state: restarted
```