# install composer

## get file setup

````
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
````

## check hash

````
HASH="$(wget -q -O - https://composer.github.io/installer.sig)"

php -r "if (hash_file('SHA384', 'composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"

````

## run setup

````
php composer-setup.php --install-dir=/usr/local/bin --filename=composer
````

## remove setup file

````
rm composer-setup.php
````

## check version

````
[root@xxx ~]# composer -V
Composer version 2.5.2 2023-02-04 14:33:22
````

