---
created: 2024-05-17T15:48
updated: 2024-05-17T15:50
tags:
  - PHP
  - MySQL
related link: https://stackoverflow.com/questions/32728860/php-7-rc3-how-to-install-missing-mysql-pdo
---
> I am trying to setup webserver with `PHP 7 RC3` + `Nginx` on `Ubuntu 14.04` (for test purposes).
> 
> I installed Ubuntu in Vagrant using `ubuntu/trusty64` and PHP 7 RC 3 from Ondřej Surý ([https://launchpad.net/~ondrej/+archive/ubuntu/php-7.0](https://launchpad.net/~ondrej/+archive/ubuntu/php-7.0)).
> 
> I can not find the way to install `MySQL PDO` (PHP sees `PDO` class but not anything related to MySQL, like `PDO::MYSQL_ATTR_DIRECT_QUERY` etc.)
> 
> Looks like there is no lib `php7.0-mysql` (by analogy with standard `php5-mysqlnd` and `php7.0-fpm` etc. from Ondřej)
> 
> Section `PDO` in `phpinfo()`:

```coffeescript
PDO support      enabled
PDO drivers      no value
```

> How can I get it?

For thoses running Linux with apache2 you need to install php-mysql

```csharp
apt-get install php-mysql
```

or if you are running ubuntu 16.04 or higher just running the following command will be enought, no need to edit your php.ini file

```csharp
apt-get install php7.2-mysql
```

---

If you are running ubuntu 15.10 or below:

Edit your **php.ini** file, it's located at `/etc/php/[version]/apache2/php.ini` and search for **pdo_mysql** you might found something like this

```swift
;extension=pdo_mysql.so
```

Change it to this

```ini
extension=pdo_mysql.so
```

Save the file and restart apache

```protobuf
service apache2 restart
```

Check that it's available in your **phpinfo()**