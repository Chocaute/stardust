---
created: 2024-03-12T15:00
updated: 2024-03-12T15:19
tags:
  - Nextcloud
  - WebHosting
  - Debian
---
> [!NOTE] Note
> Selecting the option `Cron` in the admin menu for background jobs is not mandatory, because once cron.php is executed from the command line or cron service it will set it automatically to `Cron`.

##### With crontab:
```shell
crontab -u www-data -e
```

```
*/5  *  *  *  * php -f /var/www/nextcloud/cron.php
```

##### With systemd:
```shell
nano /etc/systemd/system/nextcloudcron.service
```

```systemd
[Unit]
Description=Nextcloud cron.php job

[Service]
User=www-data
ExecCondition=php -f /var/www/nextcloud/occ status -e
ExecStart=/usr/bin/php -f /var/www/nextcloud/cron.php
KillMode=process
```

```shell
nano /etc/systemd/system/nextcloudcron.timer
``` 

```systemd
[Unit]
Description=Run Nextcloud cron.php every 5 minutes

[Timer]
OnBootSec=5min
OnUnitActiveSec=5min
Unit=nextcloudcron.service

[Install]
WantedBy=timers.target
```

```shell
systemctl enable --now nextcloudcron.timer
```

___
##### Cron job not working: 
```shell
nano /etc/php/8.2/mods-available/apcu.ini
```

```ini
apc.enable_cli=1
```

If it still doesn't work: 
```shell
sudo -u www-data php8.0 --define apc.enable_cli=1  /var/www/nextcloud/occ  maintenance:repair
```

https://help.nextcloud.com/t/oc-memcache-apcu-not-available-for-local-cache-issue-upon-initial-setup/140117
https://help.nextcloud.com/t/occ-wont-run-with-memcache-apcu/119724/7


