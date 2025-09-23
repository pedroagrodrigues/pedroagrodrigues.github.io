---
title: Upgrading PHP version
date: 2024-01-28 13:45:22
categories: [DevOps,PHP]
tags: [php, linux, ubuntu, debian, apache, nginx, php-fpm, upgrade, packages, ondrej, devops, migration]
---



## Introduction

[PHP](https://www.php.net/manual/en/intro-whatis.php) is a widely used open-source general-purpose scripting language that is especially suited for web development and can be embedded into HTML.

There are several [versions of PHP](https://www.php.net/downloads); at the time of this document, the current version was [PHP 8.3.2](https://www.php.net/ChangeLog-8.php#8.3.2).

> Before proceeding make sure your aplications are compatible with the version of PHP you are about to install.
{: .prompt-warning}

## Step 1 - List PHP packages

> You can skip this step on a fresh install.
{: .prompt-info}

We will start by ensuring we save all the current PHP packages installed.

```bash
dpkg -l | grep php | tee packages.txt
```


This command will create a named `packages.txt`. Its content looks like this:

```terminal
ii  libapache2-mod-php8.1                 8.1.12-1+ubuntu20.04.1+deb.sury.org+1                                amd64        server-side, HTML-embedded scripting language (Apache 2 module)
ii  php                                   2:8.1+92+ubuntu20.04.1+deb.sury.org+2                                all          server-side, HTML-embedded scripting language (default)
ii  php-apcu                              5.1.21+4.0.11-8+ubuntu20.04.1+deb.sury.org+1                         amd64        APC User Cache for PHP
ii  php-bcmath                            2:8.1+92+ubuntu20.04.1+deb.sury.org+2                                all          Bcmath module for PHP [default]
ii  php-bz2                               2:8.1+92+ubuntu20.04.1+deb.sury.org+2                                all          bzip2 module for PHP [default]
ii  php-cli                               2:8.1+92+ubuntu20.04.1+deb.sury.org+2                                all          command-line interpreter for the PHP scripting language (default)
ii  php-common                            2:92+ubuntu20.04.1+deb.sury.org+2                                    all          Common files for PHP packages
ii  php-curl                              2:8.1+92+ubuntu20.04.1+deb.sury.org+2                                all          CURL module for PHP [default]
ii  php-gd                                2:8.1+92+ubuntu20.04.1+deb.sury.org+2                                all          GD module for PHP [default]
ii  php-gmp                               2:8.1+92+ubuntu20.04.1+deb.sury.org+2                                all          GMP module for PHP [default]
ii  php-igbinary                          3.2.6+2.0.8-7ubuntu1                                                 amd64        igbinary PHP serializer
ii  php-imagick                           3.7.0-2+ubuntu20.04.1+deb.sury.org+2                                 amd64        Provides a wrapper to the ImageMagick library
ii  php-intl                              2:8.1+92+ubuntu20.04.1+deb.sury.org+2                                all          Internationalisation module for PHP [default]
ii  php-json                              2:8.1+92+ubuntu20.04.1+deb.sury.org+2                                all          JSON module for PHP [default]
ii  php-ldap                              2:8.1+92+ubuntu20.04.1+deb.sury.org+2                                all          LDAP module for PHP [default]
ii  php-mbstring                          2:8.1+92+ubuntu20.04.1+deb.sury.org+2                                all          MBSTRING module for PHP [default]
ii  php-mysql                             2:8.1+92+ubuntu20.04.1+deb.sury.org+2                                all          MySQL module for PHP [default]
ii  php-pear                              1:1.10.13+submodules+notgz+2022032202-2+ubuntu20.04.1+deb.sury.org+1 all          PEAR Base System
ii  php-redis                             5.3.7+4.3.0-1+ubuntu20.04.1+deb.sury.org+2                           amd64        PHP extension for interfacing with Redis
ii  php-xml                               2:8.1+92+ubuntu20.04.1+deb.sury.org+2                                all          DOM, SimpleXML, WDDX, XML, and XSL module for PHP [default]
ii  php-zip                               2:8.1+92+ubuntu20.04.1+deb.sury.org+2                                all          Zip module for PHP [default]
ii  php8.1                                8.1.12-1+ubuntu20.04.1+deb.sury.org+1                                all          server-side, HTML-embedded scripting language (metapackage)
ii  php8.1-apcu                           5.1.21+4.0.11-8+ubuntu20.04.1+deb.sury.org+1                         amd64        APC User Cache for PHP
ii  php8.1-bcmath                         8.1.12-1+ubuntu20.04.1+deb.sury.org+1                                amd64        Bcmath module for PHP
ii  php8.1-bz2                            8.1.12-1+ubuntu20.04.1+deb.sury.org+1                                amd64        bzip2 module for PHP
ii  php8.1-cli                            8.1.12-1+ubuntu20.04.1+deb.sury.org+1                                amd64        command-line interpreter for the PHP scripting language
ii  php8.1-common                         8.1.12-1+ubuntu20.04.1+deb.sury.org+1                                amd64        documentation, examples and common module for PHP
ii  php8.1-curl                           8.1.12-1+ubuntu20.04.1+deb.sury.org+1                                amd64        CURL module for PHP
ii  php8.1-dev                            8.1.12-1+ubuntu20.04.1+deb.sury.org+1                                amd64        Files for PHP8.1 module development
ii  php8.1-gd                             8.1.12-1+ubuntu20.04.1+deb.sury.org+1                                amd64        GD module for PHP
ii  php8.1-gmp                            8.1.12-1+ubuntu20.04.1+deb.sury.org+1                                amd64        GMP module for PHP
ii  php8.1-igbinary                       3.2.6+2.0.8-7ubuntu1                                                 amd64        igbinary PHP serializer
ii  php8.1-imagick                        3.7.0-2+ubuntu20.04.1+deb.sury.org+2                                 amd64        Provides a wrapper to the ImageMagick library
ii  php8.1-imap                           8.1.12-1+ubuntu20.04.1+deb.sury.org+1                                amd64        IMAP module for PHP
ii  php8.1-intl                           8.1.12-1+ubuntu20.04.1+deb.sury.org+1                                amd64        Internationalisation module for PHP
ii  php8.1-ldap                           8.1.12-1+ubuntu20.04.1+deb.sury.org+1                                amd64        LDAP module for PHP
ii  php8.1-mbstring                       8.1.12-1+ubuntu20.04.1+deb.sury.org+1                                amd64        MBSTRING module for PHP
ii  php8.1-mysql                          8.1.12-1+ubuntu20.04.1+deb.sury.org+1                                amd64        MySQL module for PHP
ii  php8.1-opcache                        8.1.12-1+ubuntu20.04.1+deb.sury.org+1                                amd64        Zend OpCache module for PHP
ii  php8.1-phpdbg                         8.1.12-1+ubuntu20.04.1+deb.sury.org+1                                amd64        server-side, HTML-embedded scripting language (PHPDBG binary)
ii  php8.1-readline                       8.1.12-1+ubuntu20.04.1+deb.sury.org+1                                amd64        readline module for PHP
ii  php8.1-redis                          5.3.7+4.3.0-1+ubuntu20.04.1+deb.sury.org+2                           amd64        PHP extension for interfacing with Redis
ii  php8.1-soap                           8.1.12-1+ubuntu20.04.1+deb.sury.org+1                                amd64        SOAP module for PHP
ii  php8.1-xml                            8.1.12-1+ubuntu20.04.1+deb.sury.org+1                                amd64        DOM, SimpleXML, XML, and XSL module for PHP
ii  php8.1-xmlrpc                         3:1.0.0~rc3-4+ubuntu20.04.1+deb.sury.org+10                          amd64        XML-RPC servers and clients functions for PHP
ii  php8.1-zip                            8.1.12-1+ubuntu20.04.1+deb.sury.org+1                                amd64        Zip module for PHP
ii  pkg-php-tools                         1.42build1                                                           all          various packaging tools and scripts for PHP packages
```

## Step 2 - Add ondrej/php repository

The newest version of PHP is not usually available to download from any current Debian or Ubuntu software repositories.

### Debian
```bash
sudo apt install apt-transport-https
sudo curl -sSLo /usr/share/keyrings/deb.sury.org-php.gpg https://packages.sury.org/php/apt.gpg
sudo sh -c 'echo "deb [signed-by=/usr/share/keyrings/deb.sury.org-php.gpg] https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list'
sudo apt update
```

### Ubuntu
```bash
sudo add-apt-repository ppa:ondrej/php # Press enter when prompted.
sudo apt update
```


## Step 3 - Install new PHP

Once you have the repositories for PHP you can install the new versions. At this time we are installing `version 8.3`.

```bash
sudo apt install php8.3-common php8.3-cli php8.3-{curl,bz2,mbstring,intl}
```

> Here, the packages.txt we've created early may come into use.
{: .prompt-tip}

I would recommend doing it manually but you can use this to install everything:
```bash
sed -n 's/php8.1/php8.3/gp' packages.txt | awk '{print $2}' | xargs sudo apt install
```

> In this case, we are moving php8.1 to php8.3; you must adapt to your case. 
{: .prompt-info}

## Step 4 - Apache Integration

In most use cases, PHP is integrated with a web server. Integrating with PHP-FPM over Fast CGI protocol is the most common approach, while it is also possible to integrate PHP with other SAPIs.

Apache web server
When installing `php8.3-fpm` package, if Apache web server (`apache2`) is present, there will be a new `php8.3-fpm.conf` file that makes it convenient to toggle PHP 8.3 integration:

```bash
sudo a2enconf php8.3-fpm
sudo a2disconf php8.2-fpm # When upgrading from an older PHP version
sudo systemctl restart apache2
```

When Apache is configured to run PHP as an Apache module (commonly called `mod_php` or `mod_php8`), install `libapache2-mod-php8.3` package instead of `php8.3-fpm`:

```bash
sudo apt install libapache2-mod-php8.3
sudo a2enmod php8.3
sudo a2dismod php8.2 # When upgrading from an older PHP version
sudo systemctl restart apache2
```

**Nginx, Caddy, Litespeed, and other servers over Fast CGI**

The `php8.3-fpm` installs PHP-FPM and registers a `systemd` service for PHP 8.3 FPM at socket address `/run/php/php8.3-fpm.sock`.

For web servers that integrate with PHP over Fast CGI, change/configure the UNIX socket path to this UNIX socket address.

For example, on Nginx, this involves changing the `fastcgi_pass` directive:

```terminal
 fastcgi_pass unix:/run/php/php8.1-fpm.sock;
 fastcgi_pass unix:/run/php/php8.2-fpm.sock;
 ```

## Step 5 - Migrate Configuration

When using PHP-FPM, make sure to replicate the correct number of FPM processes and process models.

`phpenmod` and `phpdismod` scripts provide continent toggles for PHP modules. For example, the following disables the `phar` extension for FPM on PHP 8.3:

```bash
sudo phpdismod -v 8.3 -s fpm phar 
After making changes, restart PHP 8.3-FPM:
```

sudo systemctl restart php8.3-fpm

## Step 6 - Remove Old Versions


Run `apt purge` with the PHP version prefix. For example, the following removes the packages and configuration for PHP 8.2:

```bash
sudo apt purge php8.2*
```

## Conclusion

This tutorial is a comprehensive guide for users seeking to upgrade their PHP version to the latest release, PHP 8.3, on Debian or Ubuntu systems. By meticulously outlining the steps involved, from listing and saving current PHP packages to seamlessly integrating with popular web servers like Apache, Nginx, Caddy, and Litespeed, the tutorial ensures a smooth transition to the new PHP version.

Including specific commands, explanations, and alternative approaches provides flexibility for users with varying preferences and system configurations. Furthermore, the tutorial emphasizes the importance of migrating configurations and efficiently removing old PHP versions to maintain a clean and up-to-date environment.

Whether you are a web developer or system administrator, this tutorial equips you with the knowledge and tools necessary for a successful PHP upgrade, empowering you to leverage the latest features and enhancements offered by PHP 8.3. By following these steps, users can ensure their web applications' continued compatibility and optimal performance in an ever-evolving technological landscape.