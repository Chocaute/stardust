---
created: 2024-03-15T08:31
updated: 2024-03-15T12:49
tags:
  - Ansible
  - Linux
  - Automation
---
```yaml
    - name: Update apt package cache
      apt:
        update_cache: yes

    - name: Perform a distribution upgrade
      apt:
        upgrade: dist
        autoremove: yes
        autoclean: yes
```

