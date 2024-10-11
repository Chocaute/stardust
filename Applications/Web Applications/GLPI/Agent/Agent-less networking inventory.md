---
created: 2024-06-04T15:53
updated: 2024-06-04T17:29
tags:
  - GLPI
  - GLPI-AGENT
  - NetworkDiscovery
  - Linux
  - Debian
related link: https://glpi-agent.readthedocs.io/en/latest/tasks/network-discovery.html
---
```bash
wget https://github.com/glpi-project/glpi-agent/releases/download/1.9/glpi-agent-task-network_1.9-1_all.deb
```

```bash
apt install libcrypt-des-perl libdigest-hmac-perl libnet-snmp-perl libnet-nbname-perl
```

```bash
dpkg --install glpi-agent-task-network_1.9-1_all.deb
```

```bash
glpi-netdiscovery
```

```bash
glpi-netdiscovery --first=192.168.2.1 --last=192.168.3.254 --debug
```

```bash
glpi-injector -v -d netdiscovery/ --url https://192.168.3.2/ --no-ssl-check
```


