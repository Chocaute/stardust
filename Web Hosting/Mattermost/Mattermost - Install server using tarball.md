---
created: 2024-02-29T09:45
updated: 2024-02-29T09:52
tags:
  - Mattermost
  - Debian
  - Slack
  - Chat
related link: https://docs.mattermost.com/install/install-tar.html
---
```shell
wget https://releases.mattermost.com/9.5.1/mattermost-9.5.1-linux-amd64.tar.gz
```

```shell
tar -xvzf mattermost*.gz
```

```shell
sudo mv mattermost /opt
```

```shell
sudo mkdir /opt/mattermost/data
```

```shell
sudo useradd --system --user-group mattermost
```

```shell
sudo chown -R mattermost:mattermost /opt/mattermost
```

```shell
sudo chmod -R g+w /opt/mattermost
```

```shell
sudo touch /lib/systemd/system/mattermost.service
```

```
[Unit]
Description=Mattermost
After=network.target
After=postgresql.service
BindsTo=postgresql.service

[Service]
Type=notify
ExecStart=/opt/mattermost/bin/mattermost
TimeoutStartSec=3600
KillMode=mixed
Restart=always
RestartSec=10
WorkingDirectory=/opt/mattermost
User=mattermost
Group=mattermost
LimitNOFILE=49152

[Install]
WantedBy=multi-user.target
```

```shell
sudo cp /opt/mattermost/config/config.json /opt/mattermost/config/config.defaults.json
```

```shell
sudo nano /opt/mattermost/config/config.json
```

- Set `DriverName` to `"postgres"`. This is the default and recommended database for all Mattermost installations.
    
- Set `DataSource` to `"postgres://mmuser:<mmuser-password>@<host-name-or-IP>:5432/mattermost?sslmode=disable&connect_timeout=10"` replacing `mmuser`, `<mmuser-password>`, `<host-name-or-IP>`, and `mattermost` with your database name.
    
- Set your `"SiteURL"`: The domain name for the Mattermost application (e.g. `https://mattermost.example.com`).

```shell
sudo systemctl start mattermost
```

```shell
sudo systemctl enable mattermost.service
```
