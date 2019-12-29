# (09) Configure Nova#3

	
Install OpenStack Compute Service (Nova).
This example is based on the emvironment like follows.
```
                  eth0|10.0.0.30 
          +-----------+-----------+
          |    [ Control Node ]   |
          |                       |
          |  MariaDB    RabbitMQ  |
          |  Memcached  httpd     |
          |  Keystone   Glance    |
          |   Nova API,Compute    |
          +-----------------------+
```
[1]	
Install KVM HyperVisor on Compute Host, refer to here.
It's unnecessarry to set Bridge networking on the section [3] of the link.
[2]	Install Nova Compute.
```
root@dlp ~(keystone)# apt -y install nova-compute-kvm
```
[3]	In addition to basic settings of Nova, add following settings.
```
root@dlp ~(keystone)# vi /etc/nova/nova.conf
# add follows
# enable VNC
[vnc]
enabled = True
server_listen = 0.0.0.0
server_proxyclient_address = 10.0.0.30
novncproxy_base_url = http://10.0.0.30:6080/vnc_auto.html 
[4]	Start Nova Compute service.
root@dlp ~(keystone)# systemctl restart nova-compute
# discover Compute Node
root@dlp ~(keystone)# su -s /bin/bash nova -c "nova-manage cell_v2 discover_hosts"
# show status
root@dlp ~(keystone)# openstack compute service list
+----+------------------+---------------+----------+---------+-------+----------------------------+
| ID | Binary           | Host          | Zone     | Status  | State | Updated At                 |
+----+------------------+---------------+----------+---------+-------+----------------------------+
|  1 | nova-consoleauth | dlp.srv.world | internal | enabled | up    | 2018-06-13T05:42:08.000000 |
|  2 | nova-scheduler   | dlp.srv.world | internal | enabled | up    | 2018-06-13T05:42:07.000000 |
|  3 | nova-conductor   | dlp.srv.world | internal | enabled | up    | 2018-06-13T05:42:08.000000 |
|  6 | nova-compute     | dlp.srv.world | nova     | enabled | up    | 2018-06-13T05:42:09.000000 |
+----+------------------+---------------+----------+---------+-------+----------------------------+```