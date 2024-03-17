---
created: 2024-03-15T12:47
updated: 2024-03-15T12:48
tags:
  - Ansible
  - Linux
  - Automation
---
```yml
    - name: Remove root SSH key entry
      file:
        path: /root/.ssh/authorized_keys
        state: absent
      notify:
        - restart SSH service

    - name: Disable root password
      user:
        name: root
        password: '*'

    - name: Set root user shell to /sbin/nologin
      lineinfile:
        path: /etc/passwd
        regexp: '^root:'
        line: 'root:x:0:0:root:/root:/sbin/nologin'

  handlers:
    - name: restart SSH service
      service:
        name: sshd
        state: restarted
```