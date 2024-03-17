---
created: 2024-02-26T12:19
updated: 2024-03-11T19:55
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

```yml
- name: Enable mkhomedir using pam-auth-update
  hosts: bookworm2
  tasks:
    - name: Run pam-auth-update command
      become: yes
      become_user: root
      command: pam-auth-update --enable mkhomedir
```

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