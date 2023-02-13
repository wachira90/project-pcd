# project-pcd
project pcd

## create user database


````sql
CREATE USER 'user'@'%' IDENTIFIED BY 'xxxx';
GRANT ALL PRIVILEGES ON *.* TO 'user'@'%' IDENTIFIED BY "xxxx";
FLUSH PRIVILEGES;

SELECT user, host, password, plugin FROM mysql.user; 
````

## firewall 

````bash
systemctl status firewalld
systemctl stop firewalld
systemctl disable firewalld
````
