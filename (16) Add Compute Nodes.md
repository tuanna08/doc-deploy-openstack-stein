# (16) Add Compute Nodes

	
Add Compute Nodes to run more instances.
This example is based on the emvironment like follows.
```
     ------------+-----------------------------+------------
                 |                             |
             eth0|10.0.0.30                eth0|10.0.0.51
     +-----------+-----------+     +-----------+-----------+
     |    [ Control Node ]   |     |    [ Compute Node ]   |
     |                       |     |                       |
     |  MariaDB    RabbitMQ  |     |        Libvirt        |
     |  Memcached  httpd     |     |      Nova Compute     |
     |  Keystone   Glance    |     |                       |
     |  Nova API             |     |                       |
     +-----------------------+     +-----------------------+
     ```

[1]	
Install KVM Hypervisor on Compute Node, refer to here.
It's unnecessarry to set Bridge networking on the section [3] of the link.
[2]	Install Nova-Compute.
```
root@node01:~# apt -y install nova-compute-kvm
```
[3]	Configure Nova.
```
root@node01:~# mv /etc/nova/nova.conf /etc/nova/nova.conf.org
root@node01:~# vi /etc/nova/nova.conf
# create new
[DEFAULT]
# define own IP address
my_ip = 10.0.0.51
state_path = /var/lib/nova
enabled_apis = osapi_compute,metadata
log_dir = /var/log/nova
# RabbitMQ connection info
transport_url = rabbit://openstack:password@10.0.0.30

[api]
auth_strategy = keystone

# enable VNC
[vnc]
enabled = True
server_listen = 0.0.0.0
server_proxyclient_address = $my_ip
novncproxy_base_url = http://10.0.0.30:6080/vnc_auto.html

# Glance connection info
[glance]
api_servers = http://10.0.0.30:9292

[oslo_concurrency]
lock_path = $state_path/tmp

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

[wsgi]
api_paste_config = /etc/nova/api-paste.ini

root@node01:~# chmod 640 /etc/nova/nova.conf
root@node01:~# chgrp nova /etc/nova/nova.conf
root@node01:~# systemctl restart nova-compute
```
[4]	Make sure the status of Nova services on Controle Node like here. If all State is [up], they are running normally.
```
# discover Compute Nodes
root@dlp ~(keystone)# su -s /bin/bash nova -c "nova-manage cell_v2 discover_hosts"
root@dlp ~(keystone)# openstack compute service list
+----+------------------+------------------+----------+---------+-------+----------------------------+
| ID | Binary           | Host             | Zone     | Status  | State | Updated At                 |
+----+------------------+------------------+----------+---------+-------+----------------------------+
|  1 | nova-consoleauth | dlp.srv.world    | internal | enabled | up    | 2018-06-15T04:52:08.000000 |
|  2 | nova-scheduler   | dlp.srv.world    | internal | enabled | up    | 2018-06-15T04:52:05.000000 |
|  3 | nova-conductor   | dlp.srv.world    | internal | enabled | up    | 2018-06-15T04:52:05.000000 |
|  6 | nova-compute     | dlp.srv.world    | nova     | enabled | up    | 2018-06-15T04:52:04.000000 |
|  7 | nova-compute     | node01.srv.world | nova     | enabled | up    | 2018-06-15T04:52:08.000000 |
+----+------------------+------------------+----------+---------+-------+----------------------------+
```