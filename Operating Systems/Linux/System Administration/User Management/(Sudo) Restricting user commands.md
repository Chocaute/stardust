---
created: 2024-04-02T09:36
updated: 2024-04-02T09:40
tags:
  - Linux
  - sudo
---
> [!Info] Long Story Short
> The user don't need to be a sudoer. You just need `sudo` installed on your system.

Create a special file for your not-sudoer user:
```bash
visudo -f /etc/sudoers.d/charlie_apt_update
```

Add whatever:
```plaintext
charlie ALL=(ALL) NOPASSWD: /usr/bin/apt update
```

Save and done.