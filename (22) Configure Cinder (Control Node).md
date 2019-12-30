# (22) Configure Cinder (Control Node)

Install and Configure OpenStack Block Storage (Cinder).
This example is based on the emvironment like follows.
```
     ------------+---------------------------+---------------------------+------------
                 |                           |                           |
             eth0|10.0.0.30              eth0|10.0.0.50              eth0|10.0.0.51
     +-----------+-----------+   +-----------+-----------+   +-----------+-----------+
     |    [ Control Node ]   |   |    [ Network Node ]   |   |    [ Compute Node ]   |
     |                       |   |                       |   |                       |
     |  MariaDB    RabbitMQ  |   |        L2 Agent       |   |        Libvirt        |
     |  Memcached  httpd     |   |        L3 Agent       |   |     Nova Compute      |
     |  Keystone   Glance    |   |     Metadata Agent    |   |        L2 Agent       |
     |  Nova API             |   |     Cinder-Volume     |   |                       |
     |  Neutron Server       |   |                       |   |                       |
     |  Metadata Agent       |   |                       |   |                       |
     |  Cinder API           |   |                       |   |                       |
     +-----------------------+   +-----------------------+   +-----------------------+
```
[1]	Add a User or Endpoint for Cinder to Keystone on Control Node.
```
# add cinder user (set in service project)
root@dlp ~(keystone)# openstack user create --domain default --project service --password servicepassword cinder
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| default_project_id  | fa8bf7aca73e44cb9bc2c9f42859857e |
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 000a4765d3e04b64b86d41d05a37e185 |
| name                | cinder                           |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

# add cinder user in admin role
root@dlp ~(keystone)# openstack role add --project service --user cinder admin
# add service entry for cinder
root@dlp ~(keystone)# openstack service create --name cinderv2 --description "OpenStack Block Storage" volumev2
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Block Storage          |
| enabled     | True                             |
| id          | be57cab68c6a48909bcebe11a0a722b9 |
| name        | cinderv2                         |
| type        | volumev2                         |
+-------------+----------------------------------+

root@dlp ~(keystone)# openstack service create --name cinderv3 --description "OpenStack Block Storage" volumev3
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Block Storage          |
| enabled     | True                             |
| id          | 096531d893e94363951ac8b4cfb9a79b |
| name        | cinderv3                         |
| type        | volumev3                         |
+-------------+----------------------------------+

# define cinder API host
root@dlp ~(keystone)# export controller=10.0.0.30
# add endpoint for cinder (v2 public)
root@dlp ~(keystone)# openstack endpoint create --region RegionOne volumev2 public http://$controller:8776/v2/%\(tenant_id\)s
+--------------+----------------------------------------+
| Field        | Value                                  |
+--------------+----------------------------------------+
| enabled      | True                                   |
| id           | bea4f885935744698be09cd5741f2803       |
| interface    | public                                 |
| region       | RegionOne                              |
| region_id    | RegionOne                              |
| service_id   | be57cab68c6a48909bcebe11a0a722b9       |
| service_name | cinderv2                               |
| service_type | volumev2                               |
| url          | http://10.0.0.30:8776/v2/%(tenant_id)s |
+--------------+----------------------------------------+

# add endpoint for cinder (v2 internal)
root@dlp ~(keystone)# openstack endpoint create --region RegionOne volumev2 internal http://$controller:8776/v2/%\(tenant_id\)s
+--------------+----------------------------------------+
| Field        | Value                                  |
+--------------+----------------------------------------+
| enabled      | True                                   |
| id           | 972485dcc98a49af85fcd09fb7a30948       |
| interface    | internal                               |
| region       | RegionOne                              |
| region_id    | RegionOne                              |
| service_id   | be57cab68c6a48909bcebe11a0a722b9       |
| service_name | cinderv2                               |
| service_type | volumev2                               |
| url          | http://10.0.0.30:8776/v2/%(tenant_id)s |
+--------------+----------------------------------------+

# add endpoint for cinder (v2 admin)
root@dlp ~(keystone)# openstack endpoint create --region RegionOne volumev2 admin http://$controller:8776/v2/%\(tenant_id\)s
+--------------+----------------------------------------+
| Field        | Value                                  |
+--------------+----------------------------------------+
| enabled      | True                                   |
| id           | 3f9a241ecd3948f98e9346b9a3f78cc8       |
| interface    | admin                                  |
| region       | RegionOne                              |
| region_id    | RegionOne                              |
| service_id   | be57cab68c6a48909bcebe11a0a722b9       |
| service_name | cinderv2                               |
| service_type | volumev2                               |
| url          | http://10.0.0.30:8776/v2/%(tenant_id)s |
+--------------+----------------------------------------+

# add endpoint for cinder (v3 public)
root@dlp ~(keystone)# openstack endpoint create --region RegionOne volumev3 public http://$controller:8776/v3/%\(tenant_id\)s
+--------------+----------------------------------------+
| Field        | Value                                  |
+--------------+----------------------------------------+
| enabled      | True                                   |
| id           | 7d442950613646c183c5bd6d2456d9a0       |
| interface    | public                                 |
| region       | RegionOne                              |
| region_id    | RegionOne                              |
| service_id   | 096531d893e94363951ac8b4cfb9a79b       |
| service_name | cinderv3                               |
| service_type | volumev3                               |
| url          | http://10.0.0.30:8776/v3/%(tenant_id)s |
+--------------+----------------------------------------+

# add endpoint for cinder (v3 internal)
root@dlp ~(keystone)# openstack endpoint create --region RegionOne volumev3 internal http://$controller:8776/v3/%\(tenant_id\)s
+--------------+----------------------------------------+
| Field        | Value                                  |
+--------------+----------------------------------------+
| enabled      | True                                   |
| id           | ab7752d109814de69d65c9202f0f414e       |
| interface    | internal                               |
| region       | RegionOne                              |
| region_id    | RegionOne                              |
| service_id   | 096531d893e94363951ac8b4cfb9a79b       |
| service_name | cinderv3                               |
| service_type | volumev3                               |
| url          | http://10.0.0.30:8776/v3/%(tenant_id)s |
+--------------+----------------------------------------+

# add endpoint for cinder (v3 admin)
root@dlp ~(keystone)# openstack endpoint create --region RegionOne volumev3 admin http://$controller:8776/v3/%\(tenant_id\)s
+--------------+----------------------------------------+
| Field        | Value                                  |
+--------------+----------------------------------------+
| enabled      | True                                   |
| id           | 670f97f66a9f4af19b9c928835f88200       |
| interface    | admin                                  |
| region       | RegionOne                              |
| region_id    | RegionOne                              |
| service_id   | 096531d893e94363951ac8b4cfb9a79b       |
| service_name | cinderv3                               |
| service_type | volumev3                               |
| url          | http://10.0.0.30:8776/v3/%(tenant_id)s |
+--------------+----------------------------------------+
```
[2]	Add a User and Database on MariaDB for Cinder.
```
root@dlp ~(keystone)# mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 73
Server version: 10.1.29-MariaDB-6 Ubuntu 18.04

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create database cinder;
Query OK, 1 row affected (0.00 sec)
MariaDB [(none)]> grant all privileges on cinder.* to cinder@'localhost' identified by 'password';
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> grant all privileges on cinder.* to cinder@'%' identified by 'password';
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> exit
Bye
```
[3]	Install Cinder Service.

```
root@dlp ~(keystone)# apt -y install cinder-api cinder-scheduler python-cinderclient
```
[4]	Configure Cinder.
```
root@dlp ~(keystone)# mv /etc/cinder/cinder.conf /etc/cinder/cinder.conf.org
root@dlp ~(keystone)# vi /etc/cinder/cinder.conf
# create new
[DEFAULT]
# define own IP address
my_ip = 10.0.0.30
rootwrap_config = /etc/cinder/rootwrap.conf
api_paste_confg = /etc/cinder/api-paste.ini
state_path = /var/lib/cinder
auth_strategy = keystone
# RabbitMQ connection info
transport_url = rabbit://openstack:password@10.0.0.30

# MariaDB connection info
[database]
connection = mysql+pymysql://cinder:password@10.0.0.30/cinder

# Keystone auth info
[keystone_authtoken]
www_authenticate_uri = http://10.0.0.30:5000
auth_url = http://10.0.0.30:5000
memcached_servers = 10.0.0.30:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = cinder
password = servicepassword

[oslo_concurrency]
lock_path = $state_path/tmp

root@dlp ~(keystone)# chmod 644 /etc/cinder/cinder.conf
root@dlp ~(keystone)# chown root:cinder /etc/cinder/cinder.conf
root@dlp ~(keystone)# su -s /bin/bash cinder -c "cinder-manage db sync"
root@dlp ~(keystone)# systemctl restart cinder-scheduler
# show status
root@dlp ~(keystone)# openstack volume service list
+------------------+---------------+------+---------+-------+----------------------------+
| Binary           | Host          | Zone | Status  | State | Updated At                 |
+------------------+---------------+------+---------+-------+----------------------------+
| cinder-scheduler | dlp.srv.world | nova | enabled | up    | 2018-06-18T02:56:08.000000 |
+------------------+---------------+------+---------+-------+----------------------------+
```