---
created: 2024-05-17T15:54
updated: 2024-05-17T15:58
tags:
  - PHP
  - DependancyManagement
related link: https://getcomposer.org/doc/00-intro.md
---
# Introduction
Composer is a tool for dependency management in PHP. It allows you to declare the libraries your project depends on and it will manage (install/update) them for you.

## Dependency management
Composer is **not** a package manager in the same sense as Yum or Apt are. Yes, it deals with "packages" or libraries, but it manages them on a per-project basis, installing them in a directory (e.g. `vendor`) inside your project. By default, it does not install anything globally. Thus, it is a dependency manager. It does however support a "global" project for convenience via the [global](https://getcomposer.org/doc/03-cli.md#global) command.

This idea is not new and Composer is strongly inspired by node's [npm](https://www.npmjs.com/) and ruby's [bundler](https://bundler.io/).

Suppose:

1. You have a project that depends on a number of libraries.
2. Some of those libraries depend on other libraries.

Composer:

1. Enables you to declare the libraries you depend on.
2. Finds out which versions of which packages can and need to be installed, and installs them (meaning it downloads them into your project).
3. You can update all your dependencies in one command.

See the [Basic usage](https://getcomposer.org/doc/01-basic-usage.md) chapter for more details on declaring dependencies.

___
# Download Composer

> [!WARNING]
> **Please do not redistribute the install code.** It will change with every version of the installer. Instead, please link to this page or check [how to install Composer programmatically](https://getcomposer.org/doc/faqs/how-to-install-composer-programmatically.md "See the instructions on how to install Composer programmatically").

To quickly install Composer in the current directory, run the following script in your terminal. To automate the installation, use [the guide on installing Composer programmatically](https://getcomposer.org/doc/faqs/how-to-install-composer-programmatically.md "See the instructions on how to install Composer programmatically").

```php
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === 'dac665fdc30fdd8ec78b38b9800061b4150413ff2e3b6f88543c636f7cd84f6db9189d43a81e5503cda447da73c7e5b6') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
```

This installer script will simply check some `php.ini` settings, warn you if they are set incorrectly, and then download the latest `composer.phar` in the current directory. The 4 lines above will, in order:

- Download the installer to the current directory
- Verify the installer SHA-384, which you can also [cross-check here](https://composer.github.io/pubkeys.html "Get the SHA-384 key on GitHub (external link)")
- Run the installer
- Remove the installer

Most likely, you want to put the `composer.phar` into a directory on your PATH, so you can simply call `composer` from any directory (_Global install_), using for example:

```shell
sudo mv composer.phar /usr/local/bin/composer
```

For details, [see the instructions on how to install Composer globally](https://getcomposer.org/doc/00-intro.md#globally).
