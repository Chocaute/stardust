---
created: 2024-03-13T16:36
updated: 2024-03-13T16:36
tags:
  - Debian
  - Bash
---
Open your Bash shell configuration file. Depending on your system, it might be `~/.bashrc`, `~/.bash_profile`, or `~/.bash_aliases`. You can open it with a text editor like `nano` or `vi`. For example:

```shell
nano ~/.bashrc
```

Add the following line at the end of the configuration file:

```shell
alias shortcut_name="bash /path/to/launch.sh"
```

Replace `/path/to/launch.sh` with the actual path to your `launch.sh` script.
Save and exit the text editor.
To apply the changes, either run the following command in your terminal:

```shell
source ~/.bashrc
```

Or simply open a new terminal session.
