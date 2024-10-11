---
created: 2024-03-13T17:10
updated: 2024-03-13T17:12
tags:
  - Debian
  - SSH
related link: https://linuxhint.com/disable_root_ssh_debian/
---
To disable ssh root access you need to edit the ssh configuration file, on Debian it is `/etc/ssh/sshd_config`, to edit it using nano text editor run:

```shell
nano /etc/ssh/sshd_config
```

[![](https://linuxhint.com/wp-content/uploads/2019/10/1-24.png)](https://linuxhint.com/wp-content/uploads/2019/10/1-24.png)

On nano you can press **_CTRL+W_** (where) and type **_PermitRoot_** to find the following line:

```
#PermitRootLogin prohibit-password
```

[![](https://linuxhint.com/wp-content/uploads/2019/10/2-24.png)](https://linuxhint.com/wp-content/uploads/2019/10/2-24.png)

To disable the root access through ssh just uncomment that line and replace **_prohibit-password_** for **_no_** like in the following image.

[![](https://linuxhint.com/wp-content/uploads/2019/10/3-23.png)](https://linuxhint.com/wp-content/uploads/2019/10/3-23.png)

After disabling the root access press **_CTRL+X_** and **_Y_** to save and exit.

The **_prohibit-password_** option prevents password login allowing only login through fall-back actions such as public keys, preventing brute force attacks.