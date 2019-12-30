# (19) Configure Neutron (Compute Node)


Configure OpenStack Network Service (Neutron).
This example is not All in One Settings like here, but Configure the 3 Nodes environment like follows. Neutron needs a plugin software, it's possible to choose it from some softwares. This example chooses ML2 plugin. ( it uses LinuxBridge under the backend )
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
     |  Nova API             |   |                       |   |                       |
     |  Neutron Server       |   |                       |   |                       |
     |  Metadata Agent       |   |                       |   |                       |
     +-----------------------+   +-----------------------+   +-----------------------+
```
[1]	Install required packages on Compute Node.
```
root@node01:~# apt -y install neutron-common neutron-plugin-ml2 neutron-plugin-linuxbridge-agent
```
[2]	Configure Compute Node.
```
root@node01:~# mv /etc/neutron/neutron.conf /etc/neutron/neutron.conf.org
root@node01:~# vi /etc/neutron/neutron.conf
# create new
[DEFAULT]
core_plugin = ml2
service_plugins = router
auth_strategy = keystone
state_path = /var/lib/neutron
allow_overlapping_ips = True
# RabbitMQ connection info
transport_url = rabbit://openstack:password@10.0.0.30

[agent]
root_helper = sudo /usr/bin/neutron-rootwrap /etc/neutron/rootwrap.conf

# Keystone auth info
[keystone_authtoken]
www_authenticate_uri = http://10.0.0.30:5000
auth_url = http://10.0.0.30:5000
memcached_servers = 10.0.0.30:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = servicepassword

[oslo_concurrency]
lock_path = $state_path/lock

root@node01:~# chmod 640 /etc/neutron/neutron.conf
root@node01:~# chgrp neutron /etc/neutron/neutron.conf
root@node01:~# vi /etc/neutron/plugins/ml2/ml2_conf.ini
# line 129: add ( it's OK with no value for [tenant_network_types] (set later if need) )
[ml2]
type_drivers = flat,vlan,vxlan
tenant_network_types =
mechanism_drivers = linuxbridge,l2population
extension_drivers = port_security
# line 262: uncomment and add
enable_security_group = True
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
# end line: uncomment
enable_ipset = True
root@node01:~# vi /etc/neutron/plugins/ml2/linuxbridge_agent.ini
# line 235: add own Ip address
local_ip = 10.0.0.51
root@node01:~# vi /etc/nova/nova.conf
# add follows into [DEFAULT] section
use_neutron = True
linuxnet_interface_driver = nova.network.linux_net.LinuxBridgeInterfaceDriver
firewall_driver = nova.virt.firewall.NoopFirewallDriver
vif_plugging_is_fatal = True
vif_plugging_timeout = 300
# add follows to the end: Neutron auth info
# the value of metadata_proxy_shared_secret is the same with the one in metadata_agent.ini
[neutron]
auth_url = http://10.0.0.30:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = servicepassword
service_metadata_proxy = True
metadata_proxy_shared_secret = metadata_secret
root@node01:~# ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
root@node01:~# systemctl restart nova-compute neutron-linuxbridge-agent
root@node01:~# systemctl enable neutron-linuxbridge-agent
```