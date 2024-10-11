---
created: 2024-03-15T12:45
updated: 2024-03-15T12:59
tags:
  - Ansible
  - Automation
  - Linux
---
```yml
- name: First setup
  hosts: 
  gather_facts: yes
  tasks:
    - name: Disable root password
      user:
        name: root
        password: '*'

    - name: Set root user shell to /sbin/nologin
      lineinfile:
        path: /etc/passwd
        regexp: '^root:'
        line: 'root:x:0:0:root:/root:/sbin/nologin'

    - name: Remove root SSH key entry
      file:
        path: /root/.ssh/authorized_keys
        state: absent

####

    - name: Update apt package cache
      apt:
        update_cache: yes

    - name: Perform a distribution upgrade
      apt:
        upgrade: dist
        autoremove: yes
        autoclean: yes

    - name: Install packages
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

####

    - name: Run pam-auth-update command to enable mkhomedir
      become: yes
      become_user: root
      command: pam-auth-update --enable mkhomedir

    - name: Create realmd.conf file
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

####

    - name: Create local admin user
      user:
        name: local
        shell: /bin/bash
        create_home: yes
        password: "{{ '[REDACTED]' | password_hash('sha512') }}"
        update_password: always

    - name: Add local user to sudo group
      user:
        name: local
        groups: sudo
        append: yes

    - name: Append ansible control SSH key to local's authorized_keys file
      authorized_key:
        user: local
        key: "{{ lookup('file', '/root/.ssh/id_ed25519.pub') }}"

####

    - name: Ensure sshd_config.d directory exists
      file:
        path: /etc/ssh/sshd_config.d
        state: directory
        mode: '0755'

    - name: Add configuration file to sshd_config.d
      template:
        src: sshd_config.j2
        dest: /etc/ssh/sshd_config.d/disable_password_auth.conf
        owner: root
        group: root
        mode: '0644'
      notify:
        - restart SSH service

####

  handlers:
    - name: restart SSH service
      service:
        name: sshd
        state: restarted
```