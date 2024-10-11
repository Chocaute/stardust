---
created: 2024-02-22T08:36
updated: 2024-03-12T14:53
tags:
  - WebHosting
  - Nextcloud
  - Apache2
related link: 
---
#Nextcloud 
In the .ini config file of php (non-apache), if it's something else than "-1", change "memory limit" value to 512M. 

Then, in /var/www/nextcloud/.htaccess, in the "<IfModule mod_php.c>" box, add "php_value memory_limit 512M".

Then systemctl restart apache2. 

https://help.nextcloud.com/t/changing-from-php-7-4-to-8-x-php-memory-limit-below-recommended-value-hoster-all-inkl/152395