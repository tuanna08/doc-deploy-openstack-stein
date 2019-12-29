# (07) Configure Nova#1

	
Install and Configure OpenStack Compute Service (Nova).
This example is based on the environment like follows.
        eth0|10.0.0.30 
```+-----------+-----------+
|    [ Control Node ]   |
|                       |
|  MariaDB    RabbitMQ  |
|  Memcached  httpd     |
|  Keystone   Glance    |
+-----------------------+```

[1]	Add users and others for Nova in Keystone.


```
# add nova user (set in service project)
root@dlp ~(keystone)# openstack user create --domain default --project service --password servicepassword nova
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| default_project_id  | efc5c6a05d444cae80c0a80f49ba87d4 |
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 0fb2bdb8b4a149e4b2a55b4722e3825f |
| name                | nova                             |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

# add nova user in admin role
root@dlp ~(keystone)# openstack role add --project service --user nova admin
# add placement user (set in service project)
root@dlp ~(keystone)# openstack user create --domain default --project service --password servicepassword placement
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| default_project_id  | efc5c6a05d444cae80c0a80f49ba87d4 |
| domain_id           | default                          |
| enabled             | True                             |
| id                  | acd41445432449079593614c4518b019 |
| name                | placement                        |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

# add placement user in admin role
root@dlp ~(keystone)# openstack role add --project service --user placement admin
# add service entry for nova
root@dlp ~(keystone)# openstack service create --name nova --description "OpenStack Compute service" compute
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Compute service        |
| enabled     | True                             |
| id          | 3736edccab6f4a97b8fbbcc3b3698a88 |
| name        | nova                             |
| type        | compute                          |
+-------------+----------------------------------+

# add service entry for placement
root@dlp ~(keystone)# openstack service create --name placement --description "OpenStack Compute Placement service" placement
+-------------+-------------------------------------+
| Field       | Value                               |
+-------------+-------------------------------------+
| description | OpenStack Compute Placement service |
| enabled     | True                                |
| id          | 1cbff4acc0cc44eca2f127a8d09b2c14    |
| name        | placement                           |
| type        | placement                           |
+-------------+-------------------------------------+

# define keystone host
root@dlp ~(keystone)# export controller=10.0.0.30
# add endpoint for nova (public)
root@dlp ~(keystone)# openstack endpoint create --region RegionOne compute public http://$controller:8774/v2.1/%\(tenant_id\)s
+--------------+------------------------------------------+
| Field        | Value                                    |
+--------------+------------------------------------------+
| enabled      | True                                     |
| id           | fe637cc999ff4541bf3d559e72dd753f         |
| interface    | public                                   |
| region       | RegionOne                                |
| region_id    | RegionOne                                |
| service_id   | 3736edccab6f4a97b8fbbcc3b3698a88         |
| service_name | nova                                     |
| service_type | compute                                  |
| url          | http://10.0.0.30:8774/v2.1/%(tenant_id)s |
+--------------+------------------------------------------+

# add endpoint for nova (internal)
root@dlp ~(keystone)# openstack endpoint create --region RegionOne compute internal http://$controller:8774/v2.1/%\(tenant_id\)s
+--------------+------------------------------------------+
| Field        | Value                                    |
+--------------+------------------------------------------+
| enabled      | True                                     |
| id           | e79a4abf80a448a4acc525b7c249a9ef         |
| interface    | internal                                 |
| region       | RegionOne                                |
| region_id    | RegionOne                                |
| service_id   | 3736edccab6f4a97b8fbbcc3b3698a88         |
| service_name | nova                                     |
| service_type | compute                                  |
| url          | http://10.0.0.30:8774/v2.1/%(tenant_id)s |
+--------------+------------------------------------------+

# add endpoint for nova (admin)
root@dlp ~(keystone)# openstack endpoint create --region RegionOne compute admin http://$controller:8774/v2.1/%\(tenant_id\)s
+--------------+------------------------------------------+
| Field        | Value                                    |
+--------------+------------------------------------------+
| enabled      | True                                     |
| id           | fa7f322f4ab24e84bbab7136f5c0ba8a         |
| interface    | admin                                    |
| region       | RegionOne                                |
| region_id    | RegionOne                                |
| service_id   | 3736edccab6f4a97b8fbbcc3b3698a88         |
| service_name | nova                                     |
| service_type | compute                                  |
| url          | http://10.0.0.30:8774/v2.1/%(tenant_id)s |
+--------------+------------------------------------------+

# add endpoint for placement (public)
root@dlp ~(keystone)# openstack endpoint create --region RegionOne placement public http://$controller:8778
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 8d598b66f98840c6af18b29dc25a2ff0 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 1cbff4acc0cc44eca2f127a8d09b2c14 |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://10.0.0.30:8778            |
+--------------+----------------------------------+

# add endpoint for placement (internal)
root@dlp ~(keystone)# openstack endpoint create --region RegionOne placement internal http://$controller:8778
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 3cbf3d109dff4af5862bc25b43c909aa |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 1cbff4acc0cc44eca2f127a8d09b2c14 |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://10.0.0.30:8778            |
+--------------+----------------------------------+

# add endpoint for placement (admin)
root@dlp ~(keystone)# openstack endpoint create --region RegionOne placement admin http://$controller:8778
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 095045acf0dd4a89887e4fb53a2586a3 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 1cbff4acc0cc44eca2f127a8d09b2c14 |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://10.0.0.30:8778            |
+--------------+----------------------------------+```
[2]	Add a User and Database on MariaDB for Nova.
```
root@dlp ~(keystone)# mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 39
Server version: 10.1.38-MariaDB-0ubuntu0.18.04.1 Ubuntu 18.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create database nova;
Query OK, 1 row affected (0.00 sec)
MariaDB [(none)]> grant all privileges on nova.* to nova@'localhost' identified by 'password';
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> grant all privileges on nova.* to nova@'%' identified by 'password';
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> create database nova_api;
Query OK, 1 row affected (0.00 sec)
MariaDB [(none)]> grant all privileges on nova_api.* to nova@'localhost' identified by 'password';
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> grant all privileges on nova_api.* to nova@'%' identified by 'password';
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> create database nova_placement;
Query OK, 1 row affected (0.00 sec)
MariaDB [(none)]> grant all privileges on nova_placement.* to nova@'localhost' identified by 'password';
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> grant all privileges on nova_placement.* to nova@'%' identified by 'password';
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> create database nova_cell0;
Query OK, 1 row affected (0.00 sec)
MariaDB [(none)]> grant all privileges on nova_cell0.* to nova@'localhost' identified by 'password';
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> grant all privileges on nova_cell0.* to nova@'%' identified by 'password';
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> exit
Bye
```