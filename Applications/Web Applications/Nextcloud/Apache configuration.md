---
created: 2024-02-22T09:02
updated: 2024-03-26T14:05
tags:
  - Nextcloud
  - Debian
  - Apache2
  - WebHosting
related link: https://docs.nextcloud.com/server/latest/admin_manual/installation/source_installation.html#apache-web-server-configuration
---
To use the directory-based installation, put the following in your `nextcloud.conf` replacing the **Directory** and **Alias** filepaths with the filepaths appropriate for your system:

```apacheconf title="nextcloud.conf"
Alias /nextcloud "/var/www/nextcloud/"

<Directory /var/www/nextcloud/>
  Require all granted
  AllowOverride All
  Options FollowSymLinks MultiViews

  <IfModule mod_dav.c>
    Dav off
  </IfModule>
</Directory>
```

To use the virtual host installation, put the following in your `nextcloud.conf` replacing **ServerName**, as well as the **DocumentRoot** and **Directory** filepaths with values appropriate for your system:

```apacheconf
<VirtualHost *:80>
  DocumentRoot /var/www/nextcloud/
  ServerName  your.server.com

  <Directory /var/www/nextcloud/>
    Require all granted
    AllowOverride All
    Options FollowSymLinks MultiViews

    <IfModule mod_dav.c>
      Dav off
    </IfModule>
  </Directory>
</VirtualHost>
```

```shell
a2ensite nextcloud.conf
```

For Nextcloud to work correctly, we need the module `mod_rewrite`. Enable it by running:

```shell
a2enmod rewrite
```

Additional recommended modules are `mod_headers`, `mod_env`, `mod_dir` and `mod_mime`:

```shell
a2enmod headers
a2enmod env
a2enmod dir
a2enmod mime
```

```shell
service apache2 restart
```

