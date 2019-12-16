# (02) Configure Pre-Requirements

This is the exmaple of Cloud Computiong by OpenStack Stein.
Install some services that some components of OpenStack needs for system requirements on here.
This example is based on the emvironment like follows.
        eth0|10.0.0.30 
+-----------+-----------+
|    [ Control Node ]   |
|  MariaDB    RabbitMQ  |
|  Memcached            |
+-----------------------+

[1]	Install NTP Server to adjusts the date, refer to here.
[2]	Install MariaDB Server, refer to here.
[3]	Add Openstack Stein's repository.
```
root@dlp:~# apt -y install software-properties-common
root@dlp:~# add-apt-repository cloud-archive:stein
root@dlp:~# apt update
root@dlp:~# apt -y upgrade
```

[4]	Install RabbitMQ, Memcached.

```
root@dlp:~# apt -y install rabbitmq-server memcached python-pymysql
# add openstack user (set any password you like for "password")
root@dlp:~# rabbitmqctl add_user openstack password
Creating user "openstack" ...
root@dlp:~# rabbitmqctl set_permissions openstack ".*" ".*" ".*"
Setting permissions for user "openstack" in vhost "/" ...
root@dlp:~# systemctl restart rabbitmq-server
root@dlp:~# vi /etc/mysql/mariadb.conf.d/50-server.cnf
# line 29: change
bind-address = 0.0.0.0
# line 41: uncomment and change
# default value 151 is not enough on Openstack Env
max_connections = 500
# line 111: change
character-set-server = utf8
#kcollation-server = utf8mb4_general_ci
root@dlp:~# systemctl restart mariadb
root@dlp:~# vi /etc/memcached.conf
# line 35: change
-l 0.0.0.0
root@dlp:~# systemctl restart memcached
```