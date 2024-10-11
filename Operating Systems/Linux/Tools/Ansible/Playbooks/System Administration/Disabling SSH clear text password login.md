---
created: 2024-03-15T12:49
updated: 2024-03-15T12:50
tags:
  - Ansible
  - Linux
  - Automation
---
```yaml
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

  handlers:
    - name: restart SSH service
      service:
        name: sshd
        state: restarted
```