# (03) Configure Keystone#1 

Install and Configure OpenStack Identity Service (Keystone).
This example is based on the environment like follows.
        eth0|10.0.0.30 
+-----------+-----------+
|    [ Control Node ]   |
|  MariaDB    RabbitMQ  |
|  Memcached  httpd     |
|  Keystone             |
+-----------------------+

[1]	Add a User and Database on MariaDB for Keystone.

```
root@dlp:~# mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 31
Server version: 10.1.38-MariaDB-0ubuntu0.18.04.1 Ubuntu 18.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create database keystone;
Query OK, 1 row affected (0.00 sec)
MariaDB [(none)]> grant all privileges on keystone.* to keystone@'localhost' identified by 'password';
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> grant all privileges on keystone.* to keystone@'%' identified by 'password';
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> exit
Bye
```

[2]	Install Keystone.

```
root@dlp:~# apt -y install keystone python-openstackclient apache2 libapache2-mod-wsgi-py3 python-oauth2client
```
[3]	Configure Keystone.
```
root@dlp:~# vi /etc/keystone/keystone.conf
# line 476: uncomment and specify Memcache Server
memcache_servers = 10.0.0.30:11211
# line 591: change ( MariaDB connection info )
connection = mysql+pymysql://keystone:password@10.0.0.30/keystone
# line 2544: uncomment
provider = fernet
root@dlp:~# su -s /bin/bash keystone -c "keystone-manage db_sync"
# initialize Fernet key
root@dlp:~# keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
root@dlp:~# keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
# define own (keystone) host
root@dlp:~# export controller=10.0.0.30
# keystone bootstrap (set any password for [adminpassword] section)
root@dlp:~# keystone-manage bootstrap --bootstrap-password adminpassword \
--bootstrap-admin-url http://$controller:5000/v3/ \
--bootstrap-internal-url http://$controller:5000/v3/ \
--bootstrap-public-url http://$controller:5000/v3/ \
--bootstrap-region-id RegionOne
```

[4]	Configure Apache httpd.

```
root@dlp:~# vi /etc/apache2/apache2.conf
# line 70: specify server name
ServerName dlp.srv.world
root@dlp:~# systemctl restart apache2
```
