# (05) Configure Glance

	
Install and Configure OpenStack Image Service (Glance).
This example is based on the environment like follows.
        eth0|10.0.0.30 
```+-----------+-----------+
|    [ Control Node ]   |
|                       |
|  MariaDB    RabbitMQ  |
|  Memcached  httpd     |
|  Keystone   Glance    |
+-----------------------+```

[1]	Add users and others for Glance in Keystone.

```
# add glance user (set in service project)
root@dlp ~(keystone)# openstack user create --domain default --project service --password servicepassword glance
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| default_project_id  | efc5c6a05d444cae80c0a80f49ba87d4 |
| domain_id           | default                          |
| enabled             | True                             |
| id                  | b1d7108b7f554ee6ab358a231f4b0654 |
| name                | glance                           |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

# add glance user in admin role
root@dlp ~(keystone)# openstack role add --project service --user glance admin
# add service entry for glance
root@dlp ~(keystone)# openstack service create --name glance --description "OpenStack Image service" image
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Image service          |
| enabled     | True                             |
| id          | a840550dd32440ba9d9a6d1f8f515a70 |
| name        | glance                           |
| type        | image                            |
+-------------+----------------------------------+

# define keystone host
root@dlp ~(keystone)# export controller=10.0.0.30
# add endpoint for glance (public)
root@dlp ~(keystone)# openstack endpoint create --region RegionOne image public http://$controller:9292
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 660b70d58d5f4ab4bd01ac45bc4ce1e4 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | a840550dd32440ba9d9a6d1f8f515a70 |
| service_name | glance                           |
| service_type | image                            |
| url          | http://10.0.0.30:9292            |
+--------------+----------------------------------+

# add endpoint for glance (internal)
root@dlp ~(keystone)# openstack endpoint create --region RegionOne image internal http://$controller:9292
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | e181f6fb387d441d9608a58720601988 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | a840550dd32440ba9d9a6d1f8f515a70 |
| service_name | glance                           |
| service_type | image                            |
| url          | http://10.0.0.30:9292            |
+--------------+----------------------------------+

# add endpoint for glance (admin)
root@dlp ~(keystone)# openstack endpoint create --region RegionOne image admin http://$controller:9292
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | e1ba18ad64424db0917e82137b866036 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | a840550dd32440ba9d9a6d1f8f515a70 |
| service_name | glance                           |
| service_type | image                            |
| url          | http://10.0.0.30:9292            |
+--------------+----------------------------------+
```


[2]	Add a User and Database on MariaDB for Glance.

```
root@dlp ~(keystone)# mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 39
Server version: 10.1.38-MariaDB-0ubuntu0.18.04.1 Ubuntu 18.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create database glance;
Query OK, 1 row affected (0.00 sec)
MariaDB [(none)]> grant all privileges on glance.* to glance@'localhost' identified by 'password';
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> grant all privileges on glance.* to glance@'%' identified by 'password';
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> exit
Bye
```

[3]	Install Glance.

```
root@dlp ~(keystone)# apt -y install glance
```

[4]	Configure Glance.

```
root@dlp ~(keystone)# mv /etc/glance/glance-api.conf /etc/glance/glance-api.conf.org
root@dlp ~(keystone)# vi /etc/glance/glance-api.conf
# create new
[DEFAULT]
bind_host = 0.0.0.0

[glance_store]
default_store = file
filesystem_store_datadir = /var/lib/glance/images/

[database]
# MariaDB connection info
connection = mysql+pymysql://glance:password@10.0.0.30/glance

# keystone auth info
[keystone_authtoken]
www_authenticate_uri = http://10.0.0.30:5000
auth_url = http://10.0.0.30:5000
memcached_servers = 10.0.0.30:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = glance
password = servicepassword

[paste_deploy]
flavor = keystone

root@dlp ~(keystone)# chmod 644 /etc/glance/glance-api.conf
root@dlp ~(keystone)# chown glance. /etc/glance/glance-api.conf
root@dlp ~(keystone)# su -s /bin/bash glance -c "glance-manage db_sync"
root@dlp ~(keystone)# systemctl restart glance-api
```