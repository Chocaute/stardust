---
created: 2024-02-29T11:47
updated: 2024-03-12T15:25
tags:
  - Nextcloud
  - Linux
  - WebHosting
related link: https://www.howtoforge.com/step-by-step-installing-nextcloud-on-debian-12/
---
## Prerequisites

To complete this guide, ensure you have the following:

- A Debian 12 server with at least 4 GB of memory and 2 CPUs.
- A non-root user with administrator privileges.
- A domain name pointed to the server IP address.

```shell
apt update && apt dist-upgrade && apt install -y apache2 php php-curl php-cli php-mysql php-gd php-common php-xml php-json php-intl php-pear php-imagick php-dev php-common php-mbstring php-zip php-soap php-bz2 php-bcmath php-gmp php-apcu libmagickcore-dev php-redis php-memcached php-memcache php-ldap unzip redis mariadb-server
```

```shell
nano /etc/php/8.2/apache2/php.ini
```

```ini
date.timezone = Europe/Paris

memory_limit = 512M  
upload_max_filesize = 500M  
post_max_size = 600M   
max_execution_time = 300

file_uploads = On  
allow_url_fopen = On

display_errors = Off  
output_buffering = Off

zend_extension=opcache

opcache.enable = 1  
opcache.interned_strings_buffer = 64  
opcache.max_accelerated_files = 10000  
opcache.memory_consumption = 512  
opcache.save_comments = 1  
opcache.revalidate_freq = 1
```

```shell
systemctl restart apache2
```

```shell
systemctl mariadb-secure-installation
```

```shell
mariadb
```

```sql
CREATE DATABASE nextcloud_db;  
CREATE USER nextclouduser@localhost IDENTIFIED BY 'StrongPassword';
GRANT ALL PRIVILEGES ON nextcloud_db.* TO nextclouduser@localhost;  
FLUSH PRIVILEGES;
SHOW GRANTS FOR nextclouduser@localhost;
EXIT;
```

```shell
cd /var/www/
```

```shell
curl -o nextcloud.zip https://download.nextcloud.com/server/releases/latest.zip
```

```shell
unzip nextcloud.zip
```

```shell
chown -R www-data:www-data nextcloud
```

```shell
nano /etc/apache2/sites-available/nextcloud.conf
```

```apacheconf
<VirtualHost *:80>  
    ServerName nextcloud.hwdomain.io  
    DocumentRoot /var/www/nextcloud/  
  
    # log files  
    ErrorLog /var/log/apache2/files.hwdomain.io-error.log  
    CustomLog /var/log/apache2/files.hwdomain.io-access.log combined  
  
    <Directory /var/www/nextcloud/>  
        Options +FollowSymlinks  
        AllowOverride All  
  
        <IfModule mod_dav.c>  
            Dav off  
        </IfModule>  
  
        SetEnv HOME /var/www/nextcloud  
        SetEnv HTTP_HOME /var/www/nextcloud  
    </Directory>  
</VirtualHost>
```

```shell
a2ensite nextcloud.conf
```

```shell
systemctl restart apache2
```

```shell
nano /var/www/nextcloud/config/config.php
```

```php
  'default_phone_region' => 'FR',
  'memcache.local' => '\OC\Memcache\APCu',
  'memcache.distributed' => '\OC\Memcache\Redis',
  'memcache.locking' => '\OC\Memcache\Redis',
  'redis' => [
       'host' => 'localhost',
       'port' => 6379,
  ],
  'maintenance_window_start' => 1,
```

You'll also need to setup:
- [[Nextcloud - Cron jobs]]
- [[Nextcloud - Proxies - cardav & caldav (Service Discovery)]]
- [[Nextcloud - Proxies - HSTS]]





