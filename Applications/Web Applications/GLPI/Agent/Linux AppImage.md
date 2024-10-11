---
created: 2023-12-21T16:47
updated: 2024-06-04T17:49
tags:
  - GLPI
  - GLPI-AGENT
  - Debian
  - WebHosting
related link: https://glpi-agent.readthedocs.io/en/latest/installation/linux-appimage.html
---
Get the lastest version on https://github.com/glpi-project/glpi-agent/releases

Make it executable:
```shell
chmod +x ./glpi-agent-*
./glpi-agent-*
```

Can be run on almost any linux distrib.

My default install parameters:
```shell
./glpi-agent-1.6.1-x86_64.AppImage --install --local=/tmp/
```

Then, at any time:
```shell
glpi-agent > /tmp/glpi-agent.dump.xml
```
