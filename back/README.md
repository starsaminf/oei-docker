# Docker Container for running apache on CentOS 6.7
The docker container is based on CentOS 6.7.
Apache v2.2.15 has been installed to support php v5.6

### Installed PHP Packages

- php
- php-devel
- php-mysql
- php-common
- php-mbstring
- php-cli
- php-pear
- php-mcrypt
- php-process
- php-soap
- php-gd
- php-opcache
- php-pdo
- php-pear
- php-pecl-apcu
- php-pecl-memcache
- php-tidy
- php-xml
- php-xmlrpc
from remi-php56,remi

### Docker Installation
Install [Docker](https://docs.docker.com/installation/) on your favourite distro and run the container

### Customize apache conf files

Customize conf file before build Dockerfile
default virtual.conf is
```bash
    ServerName d.er
    RewriteEngine On

    <Directory /var/www/html>
        AllowOverride All
    </Directory>
```

### Run Container
Run the below commands to run the container

```bash
docker pull kotamat/laravel

docker run --name <any_name> -e 'TIMEZONE=Asia/Tokyo' -d -p <host_http_port>:80 -p <host_https_port>:443 -v <path_of_website_files>:/var/www/html kotamat/laravel
```
