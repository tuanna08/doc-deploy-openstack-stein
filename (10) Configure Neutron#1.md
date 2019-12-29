# (10) Configure Neutron#1

	
Configure OpenStack Network Service (Neutron).
This example is based on the emvironment like follows.
        eth0|10.0.0.30 
```+-----------+-----------+
|    [ Control Node ]   |
|                       |
|  MariaDB    RabbitMQ  |
|  Memcached  httpd     |
|  Keystone   Glance    |
|  Nova API             |
+-----------------------+```

[1]	Add user or service for Neutron on Keystone Server.

```
# add neutron user (set in service project)
root@dlp ~(keystone)# openstack user create --domain default --project service --password servicepassword neutron
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| default_project_id  | efc5c6a05d444cae80c0a80f49ba87d4 |
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 2fa298c77c1d4b73a30b18e4c600d76f |
| name                | neutron                          |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

# add neutron user in admin role
root@dlp ~(keystone)# openstack role add --project service --user neutron admin
# add service entry for neutron
root@dlp ~(keystone)# openstack service create --name neutron --description "OpenStack Networking service" network
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Networking service     |
| enabled     | True                             |
| id          | cae18783aee749e791e0e9dd26201607 |
| name        | neutron                          |
| type        | network                          |
+-------------+----------------------------------+

# define keystone host
root@dlp ~(keystone)# export controller=10.0.0.30
# add endpoint for neutron (public)
root@dlp ~(keystone)# openstack endpoint create --region RegionOne network public http://$controller:9696
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | bd845460e1da4a219cdb5d944a3b3ea2 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | cae18783aee749e791e0e9dd26201607 |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://10.0.0.30:9696            |
+--------------+----------------------------------+

# add endpoint for neutron (internal)
root@dlp ~(keystone)# openstack endpoint create --region RegionOne network internal http://$controller:9696
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 6c60186feace404b92592655b9534482 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | cae18783aee749e791e0e9dd26201607 |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://10.0.0.30:9696            |
+--------------+----------------------------------+

# add endpoint for neutron (admin)
root@dlp ~(keystone)# openstack endpoint create --region RegionOne network admin http://$controller:9696
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 4ba82ecff4d243fd8da68a7bbf77675b |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | cae18783aee749e791e0e9dd26201607 |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://10.0.0.30:9696            |
+--------------+----------------------------------+```

[2]	Add a User and Database on MariaDB for Neutron.
```
root@dlp ~(keystone)# mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 97
Server version: 10.1.38-MariaDB-0ubuntu0.18.04.1 Ubuntu 18.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create database neutron_ml2;
Query OK, 1 row affected (0.00 sec)
MariaDB [(none)]> grant all privileges on neutron_ml2.* to neutron@'localhost' identified by 'password';
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> grant all privileges on neutron_ml2.* to neutron@'%' identified by 'password';
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> exit
Bye```