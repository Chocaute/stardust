---
created: 2024-03-15T13:07
updated: 2024-10-08T14:00
tags:
  - Ansible
  - Linux
  - Automation
  - AD
---
Note for future me:
> In case of "realm: Couldn't join realm: Insufficient permissions to join the domain", you need to modify or add `/etc/krb5.conf` with the following content :
```
[libdefaults]
default_realm = YOUR.DOMAIN.COM
rdns = false
```
> My theory is that the issue could stems from a mismatch between forward and reverse DNS lookups for your domain controller:
> 
> 1. Forward lookup: `DC1.YOUR.DOMAIN.COM` → 192.168.1.10
> 2. Reverse lookup: 192.168.1.10 → `YOUR.DOMAIN.COM`
> 
> This inconsistency causes Kerberos authentication to fail when rdns=true, because the reverse lookup doesn't match the original hostname used. Setting rdns=false bypasses this check, allowing authentication to succeed, but at the cost of skipping a security verification step.

```yml
- name: Join the tomraud.fr AD
  hosts: bookworm2
  gather_facts: yes
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

    - name: Run pam-auth-update command
      become: yes
      become_user: root
      command: pam-auth-update --enable mkhomedir

    - name: Create realmd.conf file with OS info
      template:
        src: templates/realmd.conf.j2
        dest: /etc/realmd.conf
      vars:
        os_name: "{{ ansible_distribution }}"
        os_version: "{{ ansible_distribution_version }}"

    - name: Join the realm using realm join command
      expect:
        command: realm join tomraud.fr --user=administrateur
        responses:
          "Password for administrateur:": [REDACTED]

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

    - name: Add sudoers configuration to nsswitch.conf
      blockinfile:
        path: /etc/sssd/sssd.conf
        block: |
          ldap_sudo_search_base = ou=DevRules,ou=sudoRules,ou=Linux,ou=Postes de travail,dc=tomraud,dc=fr

	- name: Restart sssd service
      systemd:
        name: sssd
        state: restarted
```