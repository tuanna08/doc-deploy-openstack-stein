# (08) Configure Nova#2

	
Install and Configure OpenStack Compute Service (Nova).
This example is based on the environment like follows.
```
                  eth0|10.0.0.30 
          +-----------+-----------+
          |    [ Control Node ]   |
          |                       |
          |  MariaDB    RabbitMQ  |
          |  Memcached  httpd     |
          |  Keystone   Glance    |
          |  Nova API             |
          +-----------------------+
```
[1]	Install Nova.
```
root@dlp ~(keystone)# apt -y install nova-api nova-placement-api nova-conductor nova-consoleauth nova-scheduler nova-novncproxy python-novaclient
```
[2]	Configure Nova.

```
root@dlp ~(keystone)# mv /etc/nova/nova.conf /etc/nova/nova.conf.org
root@dlp ~(keystone)# vi /etc/nova/nova.conf
# create new
[DEFAULT]
# define own IP
my_ip = 10.0.0.30
state_path = /var/lib/nova
enabled_apis = osapi_compute,metadata
log_dir = /var/log/nova
# RabbitMQ connection info
transport_url = rabbit://openstack:password@10.0.0.30

[api]
auth_strategy = keystone

# Glance connection info
[glance]
api_servers = http://10.0.0.30:9292

[oslo_concurrency]
lock_path = $state_path/tmp

# MariaDB connection info
[api_database]
connection = mysql+pymysql://nova:password@10.0.0.30/nova_api

[database]
connection = mysql+pymysql://nova:password@10.0.0.30/nova

# Keystone auth info
[keystone_authtoken]
www_authenticate_uri = http://10.0.0.30:5000
auth_url = http://10.0.0.30:5000
memcached_servers = 10.0.0.30:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = servicepassword

[placement]
auth_url = http://10.0.0.30:5000
os_region_name = RegionOne
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = placement
password = servicepassword

[placement_database]
connection = mysql+pymysql://nova:password@10.0.0.30/nova_placement

[wsgi]
api_paste_config = /etc/nova/api-paste.ini

root@dlp ~(keystone)# chmod 640 /etc/nova/nova.conf
root@dlp ~(keystone)# chgrp nova /etc/nova/nova.conf
```
[3]	Add Data into Database and start Nova services.
```
It doesn't need to care the messages "deprecated ***" when sync DB.
root@dlp ~(keystone)# su -s /bin/bash nova -c "nova-manage api_db sync"
root@dlp ~(keystone)# su -s /bin/bash nova -c "nova-manage cell_v2 map_cell0"
root@dlp ~(keystone)# su -s /bin/bash nova -c "nova-manage db sync"
root@dlp ~(keystone)# su -s /bin/bash nova -c "nova-manage cell_v2 create_cell --name cell1"
root@dlp ~(keystone)# systemctl restart apache2
root@dlp ~(keystone)# for service in api conductor scheduler consoleauth novncproxy; do
systemctl restart nova-$service
done
# show status
root@dlp ~(keystone)# openstack compute service list
+----+------------------+---------------+----------+---------+-------+----------------------------+
| ID | Binary           | Host          | Zone     | Status  | State | Updated At                 |
+----+------------------+---------------+----------+---------+-------+----------------------------+
|  1 | nova-consoleauth | dlp.srv.world | internal | enabled | up    | 2018-06-13T05:32:33.000000 |
|  2 | nova-scheduler   | dlp.srv.world | internal | enabled | up    | 2018-06-13T05:32:23.000000 |
|  3 | nova-conductor   | dlp.srv.world | internal | enabled | up    | 2018-06-13T05:32:23.000000 |
+----+------------------+---------------+----------+---------+-------+----------------------------+
```