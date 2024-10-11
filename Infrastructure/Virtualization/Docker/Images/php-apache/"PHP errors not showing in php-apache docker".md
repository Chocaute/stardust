---
created: 2024-05-17T15:36
updated: 2024-05-17T15:39
tags:
  - Docker
  - PHP
  - Apache2
related link: https://stackoverflow.com/questions/76231063/php-errors-not-showing-in-php-apache-docker
---
> I am trying to get the error logs out of my php docker. I found that the error.log file was going nowhere, hence modified my dockerfile like this:

```php
FROM php:7.4-apache
RUN a2enmod rewrite
RUN docker-php-ext-install mysqli pdo pdo_mysql; \
    docker-php-ext-enable mysqli pdo pdo_mysql;

# Redirect apache log files to stdout, see https://github.com/moby/moby/issues/19616
RUN ln -sf /proc/1/fd/1 /var/log/apache2/access.log
RUN ln -sf /proc/1/fd/1 /var/log/apache2/error.log
```

> Resulting in the files below:

```php
# ls -l /var/log/apache2/
total 0
lrwxrwxrwx 1 root     root     12 May 11 18:25 access.log -> /proc/1/fd/1
lrwxrwxrwx 1 root     root     12 May 11 18:25 error.log -> /proc/1/fd/1
lrwxrwxrwx 1 www-data www-data 11 Nov 15 04:17 other_vhosts_access.log -> /dev/stdout
```

> That part is now working: running `echo "test" > /var/log/apache2/error.log` does write test to the docker logs. Plus, file permissions are not restrictive (see [https://stackoverflow.com/a/12566881/3519951](https://stackoverflow.com/a/12566881/3519951)).
> 
> I added a crashing call, `dfgdfga.test()`, but the error doesn't show up. Same if I create a log file instead of redirecting to docker output. It remains empty.
___
# Answer

While I was reviewing the question I figured it out. Sharing here in case it might help.

Running `php -i | grep log` I found this:

```php
log_errors => Off => Off
```

It's a surprising default in my opinion, but more than enough to explain the absence of logs. So I created a small ini file ([inspired from this article](https://mediatemple.net/community/products/dv/360020711631/how-to-enable-php-error-logging-via-php.ini)) to pipe it straight to the docker logs:

```php
; log PHP errors to a file. E_ALL=32767
log_errors = on
error_reporting = 32767
error_log = /proc/1/fd/1
```

And copied it in dockerfile like this:

```php
COPY logging.ini /usr/local/etc/php/conf.d/logging.ini
```
