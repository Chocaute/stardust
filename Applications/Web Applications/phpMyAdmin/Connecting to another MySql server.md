---
created: 2024-05-17T16:01
updated: 2024-05-17T16:10
tags:
  - phpMyAdmin
  - SQL
related link: https://serverfault.com/questions/402694/configure-phpmyadmin-to-connect-to-another-mysql-server
---
# Either

> You can enable the feature that allows specifying the server to connect to on the login page by configuring the `$cfg['AllowArbitraryServer']` directive in the phpMyAdmin configuration file. 
> 
> Add the following line to enable the arbitrary server feature:

```php
$cfg['AllowArbitraryServer'] = true;
```

# Or

> [!WARNING] This is unlikely to help.
> it is only relevant to a small geographic area, a specific moment in time, or an extraordinarily narrow situation that is not generally applicable.

> PHPMyAdmin's [configuration file](http://wiki.phpmyadmin.net/pma/config.inc.php) is located in the root directory and is called _config.inc.php_ and in order to configure the servers to connect to, you can check [this page](http://wiki.phpmyadmin.net/pma/Config/Servers) for more information.
> 
> Here's an example how the server configuration should look like:

```php
$i++;

/* First server */
$cfg['Servers'][$i]['auth_type'] = 'cookie';
$cfg['Servers'][$i]['host'] = 'mysql1.example.net';
$cfg['Servers'][$i]['connect_type'] = 'tcp';
$cfg['Servers'][$i]['compress'] = false;
$cfg['Servers'][$i]['extension'] = 'mysql';
$cfg['Servers'][$i]['AllowNoPassword'] = false;

$i++;

/* Second server */
$cfg['Servers'][$i]['auth_type'] = 'cookie';
$cfg['Servers'][$i]['host'] = 'mysql2.example.net';
$cfg['Servers'][$i]['connect_type'] = 'tcp';
$cfg['Servers'][$i]['compress'] = false;
$cfg['Servers'][$i]['extension'] = 'mysql';
$cfg['Servers'][$i]['AllowNoPassword'] = false;
```


