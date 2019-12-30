# (23) Configure Cinder (Storage Node)

	
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
[1]	Install Cinder Volume.
```
root@storage:~# apt -y install cinder-volume python-mysqldb
```
[2]	Configure Cinder Volume.
```
root@storage:~# mv /etc/cinder/cinder.conf /etc/cinder/cinder.conf.org
root@storage:~# vi /etc/cinder/cinder.conf
# create new
[DEFAULT]
# define own IP address
my_ip = 10.0.0.50
rootwrap_config = /etc/cinder/rootwrap.conf
api_paste_confg = /etc/cinder/api-paste.ini
state_path = /var/lib/cinder
auth_strategy = keystone
# RabbitMQ connection info
transport_url = rabbit://openstack:password@10.0.0.30
# Glance connection info
glance_api_servers = http://10.0.0.30:9292
# OK with empty value now
enabled_backends =

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

root@storage:~# chmod 644 /etc/cinder/cinder.conf
root@storage:~# chown root:cinder /etc/cinder/cinder.conf
root@storage:~# systemctl restart cinder-volume
```