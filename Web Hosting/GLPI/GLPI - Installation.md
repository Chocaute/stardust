---
created: 2023-12-08T17:08
updated: 2024-03-11T19:29
tags:
  - GLPI
  - Debian
  - WebHosting
---
##### LAMP STACK + GLPI DEPEDENCIES
```shell
apt update
apt dist-upgrade -y
apt install apache2 php php8.2-fpm mariadb-server php-xml php-common php-json php-mysql php-mbstring php-curl php-gd php-intl php-zip php-bz2 php-imap php-apcu php-ldap -y 
```

##### PHP
```shell
nano /etc/php/8.2/fpm/php.ini
```

```shell
session.cookie_httponly = on
```

```shell
systemctl restart php8.2-fpm.service
```

##### Apache
```shell
nano /etc/apache2/sites-available/glpi.conf
```

```apacheconf
<VirtualHost *:80>
    ServerName glpi.localhost

    DocumentRoot /var/www/glpi/public

    # If you want to place GLPI in a subfolder of your site (e.g. your virtual host is serving multiple applications),
    # you can use an Alias directive. If you do this, the DocumentRoot directive MUST NOT target the GLPI directory itself.
    # Alias "/glpi" "/var/www/glpi/public"

    <Directory /var/www/glpi/public>
        Require all granted

        RewriteEngine On

        # Redirect all requests to GLPI router, unless file exists.
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteRule ^(.*)$ index.php [QSA,L]
    </Directory>
    
	<FilesMatch \.php$>
	    SetHandler "proxy:unix:/run/php/php8.2-fpm.sock|fcgi://localhost/"
	</FilesMatch>
</VirtualHost>
```

Changer "glpi.localhost" par le nom de domaine de l'instance GLPI.

```shell
a2dissite 000-default.conf
a2ensite glpi.conf
a2enmod rewrite
a2enmod proxy_fcgi
a2enconf php8.2-fpm
systemctl restart apache2
```

##### MariaDB
```shell
mysql_secure_installation
```
```
Enter current password for root (enter for none):# 
OK, successfully used password, moving on...
Switch to unix_socket authentication [Y/n]# n
 ... skipping.
Change the root password? [Y/n]# n
 ... skipping.
 Remove anonymous users? [Y/n]# Y
 ... Success!
Disallow root login remotely? [Y/n]# Y
 ... Success!
 Remove test database and access to it? [Y/n]# Y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!
Reload privilege tables now? [Y/n]# Y
 ... Success!
```

##### GLPI - Nouvelle DB
```shell
mysql
```
```sql
CREATE DATABASE glpi_db;
```
```sql
GRANT ALL PRIVILEGES ON glpi_db.* TO glpi_user@localhost IDENTIFIED BY "mot de passe";
```
```sql
FLUSH PRIVILEGES;
```
```sql
EXIT
```

##### GLPI - Migration DB
```shell
mysql glpi_db < glpi_db_dump.sql
```

##### GLPI - Installation
```shell
cd /tmp
wget https://github.com/glpi-project/glpi/releases/download/10.0.11/glpi-10.0.12.tgz
tar -xzvf glpi-10.0.12.tgz -C /var/www/
chown www-data /var/www/glpi/ -R
mkdir /etc/glpi
chown www-data /etc/glpi/
mv /var/www/glpi/config /etc/glpi
mkdir /var/lib/glpi
chown www-data /var/lib/glpi/
mv /var/www/glpi/files /var/lib/glpi
mkdir /var/log/glpi
chown www-data /var/log/glpi
echo -e "<?php\ndefine('GLPI_CONFIG_DIR', '/etc/glpi/');\nif (file_exists(GLPI_CONFIG_DIR . '/local_define.php')) {\n    require_once GLPI_CONFIG_DIR . '/local_define.php';\n}" > /var/www/glpi/inc/downstream.php
echo -e "<?php\ndefine('GLPI_VAR_DIR', '/var/lib/glpi/files');\ndefine('GLPI_LOG_DIR', '/var/log/glpi');" > /etc/glpi/local_define.php
```

##### GLPI - Setup
![[Pasted image 20231208170010.png]]

**Nouvelle base de données -> Installer
Migration d'une base de données -> Mettre à jour 

##### GLPI - Post-Installation
```shell
rm /var/www/glpi/install/install.php
```
