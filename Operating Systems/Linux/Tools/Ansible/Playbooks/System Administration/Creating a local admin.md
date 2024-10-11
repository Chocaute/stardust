---
created: 2024-03-15T12:48
updated: 2024-03-15T12:49
tags:
  - Ansible
  - Linux
  - Automation
---
```yaml
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
```
