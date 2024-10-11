---
created: 2024-06-04T17:49
updated: 2024-06-04T19:22
tags:
  - GLPI-AGENT
  - Linux
related link: https://glpi-agent.readthedocs.io/en/latest/installation/index.html#linux-perl-installer
---
### Linux Perl Installer


> [!WARNING] 
> The **linux installer** main requirement is the **perl** command.
> It also requires one of the following command, depending on the targeted system:
> - **dnf** for recent **RPM** based systems ;
> - **yum** for previous generation of **RPM** based systems ;
> - **apt** for **DEB** based systems like Debian & Ubuntu ;
> - **snap** for other systems supporting [Snapcraft](https://snapcraft.io/) **Snap** packages.

We also provide a dedicated linux installer which includes all the packages we build (**RPM** & **DEB**) and eventually the [snap](https://glpi-agent.readthedocs.io/en/latest/installation/index.html#snap) one. On supported distros (**DEB** & **RPM** based), the installer will also eventually try to enable third party repositories, like EPEL on CentOS if they are required.

The installer is a simple perl script. It supports various options to configure the agent during installation. If no options are specified, the default is for the installer to configure glpi-agent to not scan user home directories or profiles, to run inventory modules with a 30 minute timeout, to allow HTTP requests to port 62354 only from the GLPI server, to setup the glpi-agent as a service and to install only the Computer Inventory and Remote Inventory tasks. You can check all supported options by running:

```shell
perl glpi-agent-1.8-linux-installer.pl --help
```

or if you use the installer embedding **snap** package:

```shell
perl glpi-agent-1.8-with-snap-linux-installer.pl --help
```

If your GNU/Linux distro is not supported, you still can [install it from sources](https://glpi-agent.readthedocs.io/en/latest/installation/index.html#install-from-sources).