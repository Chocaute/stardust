---
created: 2024-05-17T19:31
updated: 2024-05-17T19:32
tags:
  - GLPI
  - WebHosting
related link: https://help.nextcloud.com/t/occ-command-not-found/54852/6
---
> You might also be interested in the [admin docs 244](https://docs.nextcloud.com/server/16/admin_manual/configuration_server/occ_command.html#run-occ-as-your-http-user) for running the OCC command - notably, it should be run as your web user (whatever user is running Apache/nginx).

> As an example, here’s how I run OCC on my Ubuntu setup:  
> `sudo -u www-data php /var/www/nextcloud/occ`  
> I could leave out the ‘php’ part, but this way I don’t have to worry about setting execute permissions.