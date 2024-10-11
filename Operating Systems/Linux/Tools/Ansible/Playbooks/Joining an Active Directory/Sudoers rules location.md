---
created: 2024-03-15T13:06
updated: 2024-03-15T13:07
tags:
  - Ansible
  - Linux
  - Automation
  - AD
---
```yml
	- name: Add sudoers configuration to nsswitch.conf
      blockinfile:
        path: /etc/sssd/sssd.conf
        block: |
          ldap_sudo_search_base = ou=DevRules,ou=sudoRules,ou=Linux,ou=Postes de travail,dc=tomraud,dc=fr
```
