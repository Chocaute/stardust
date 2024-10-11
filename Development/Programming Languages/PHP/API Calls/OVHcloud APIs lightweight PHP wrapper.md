---
created: 2024-05-17T15:59
updated: 2024-05-17T16:01
tags:
  - PHP
  - API
  - OVH
related link: https://github.com/ovh/php-ovh
---
This PHP package is a lightweight wrapper for OVHcloud APIs, the easiest way to use OVHcloud APIs in your PHP applications.

> Compatible with PHP 7.4, 8.0, 8.1, 8.2.

## Installation

Install this wrapper and integrate it inside your PHP application with [Composer](https://getcomposer.org):

```php
composer require ovh/ovh
```

## Basic usage

```php
<?php
require __DIR__ . '/vendor/autoload.php';
use \Ovh\Api;

// Api credentials can be retrieved from the urls specified in the "Supported endpoints" section below.
$ovh = new Api($applicationKey,
                $applicationSecret,
                $endpoint,
                $consumerKey);
echo 'Welcome '.$ovh->get('/me')['firstname'];
```