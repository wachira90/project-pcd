# How to Install PHP 8.2, 8.1, 8.0, 7.4 on Fedora Linux

## Import Remi PHP Repository

### Import Remi PHP Repository for Fedora 37

```bash
sudo dnf install http://rpms.remirepo.net/fedora/remi-release-37.rpm -y
```

### Import Remi PHP Repository for Fedora 36

```bash
sudo dnf install http://rpms.remirepo.net/fedora/remi-release-36.rpm -y
```

### Import Remi PHP Repository for Fedora 35

```bash
sudo dnf install http://rpms.remirepo.net/fedora/remi-release-35.rpm -y
```

#### After installing, it’s important to check the installation is successful.

```bash
dnf repolist | grep remi
```

## Enable PHP Remi Repository

```bash
dnf module list php
```


### Enable PHP 8.2 on Fedora.

```bash
sudo dnf module enable php:remi-8.2 -y
```

### Enable PHP 8.1 on Fedora.

```bash
sudo dnf module enable php:remi-8.1 -y
```

### Enable PHP 8.0 on Fedora.

```bash
sudo dnf module enable php:remi-8.0 -y
```

### Enable PHP 7.4 on Fedora.

```bash
sudo dnf module enable php:remi-7.4 -y
```

## Install PHP 8.2, 8.1, 8.0 or 7.4

### Apache (httpd) PHP:

```bash
sudo dnf install php php-cli -y
```

### Nginx PHP:

```bash
sudo dnf install php-fpm php-cli -y
```

To confirm that the installation of PHP was successful, you can run the following command after the installation process is finished.

```bash
php -v
```

If you wish, you can run the command below to acquire the most commonly used extensions for your chosen version of PHP. It’s important to note that you should remove any extensions you know will not be needed for your specific use case.

```bash
sudo dnf install php-cli php-fpm php-curl php-mysqlnd php-gd php-opcache php-zip php-intl php-common php-bcmath php-imagick php-xmlrpc php-json php-readline php-memcached php-redis php-mbstring php-apcu php-xml php-dom php-redis php-memcached php-memcache
```

Another method to install the commands above is by using the format of PHP-{extension}.

```bash
sudo dnf install php-{cli,fpm,curl,mysqlnd,gd,opcache,zip,intl,common,bcmath,imagick,xmlrpc,json,readline,memcached,redis,mbstring,apcu,xml,dom,redis,memcached,memcache}
```

The command below can be executed at any time to view the currently loaded modules.

```bash
php -m
```

If you’re interested in installing the development branch of PHP, you can use the command provided below.

```bash
sudo dnf install php-devel
```

To install additional development tools like debugging, use the command below.

```bash
sudo dnf install php-xdebug php-pcov
```

## Set Up Nginx user for PHP-FPM

The first step is to open the configuration file (www.conf) using the below command.

```bash
sudo nano /etc/php-fpm.d/www.conf
```

Once the configuration file is open, you must replace the (Apache) user and group with the (Nginx) user and group.

```bash
user = nginx
group = nginx
```

After making the necessary adjustments, it’s time to restart your PHP-FPM service.

```bash
sudo systemctl restart php-fpm
```

## Example Nginx PHP-FPM Server Block Code

```lua
location ~ \.php$ {
    try_files $uri =404;
    fastcgi_pass unix:/run/php-fpm/www.sock;
    fastcgi_index   index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
}
```

To verify that the changes made to the previous code did not cause any errors, you can use the following command to test the Nginx configuration.

```bash
sudo nginx -t
```

Example output:

```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

To complete the PHP-FPM setup, it’s necessary to restart the Nginx service.

```bash
sudo systemctl restart nginx
```
