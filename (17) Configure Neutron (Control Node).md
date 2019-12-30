# (17) Configure Neutron (Control Node)

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

[1]	Add user or service for Neutron on Keystone Server, refer to here.
[2]	Install Neutron Server on Control Host.
```
root@dlp ~(keystone)# apt -y install neutron-server neutron-metadata-agent neutron-plugin-ml2 python-neutronclient
```

[3]	Configure Neutron Server.

```
root@dlp ~(keystone)# mv /etc/neutron/neutron.conf /etc/neutron/neutron.conf.org
root@dlp ~(keystone)# vi /etc/neutron/neutron.conf
# create new
[DEFAULT]
core_plugin = ml2
service_plugins = router
auth_strategy = keystone
state_path = /var/lib/neutron
dhcp_agent_notification = True
allow_overlapping_ips = True
notify_nova_on_port_status_changes = True
notify_nova_on_port_data_changes = True
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

# MariaDB connection info
[database]
connection = mysql+pymysql://neutron:password@10.0.0.30/neutron_ml2

# Nova connection info
[nova]
auth_url = http://10.0.0.30:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = servicepassword

[oslo_concurrency]
lock_path = $state_path/tmp

root@dlp ~(keystone)# chmod 640 /etc/neutron/neutron.conf
root@dlp ~(keystone)# chgrp neutron /etc/neutron/neutron.conf
root@dlp ~(keystone)# vi /etc/neutron/metadata_agent.ini
# line 22: uncomment and specify Nova API server
nova_metadata_host = 10.0.0.30
# line 34: uncomment and specify any secret key you like
metadata_proxy_shared_secret = metadata_secret
root@dlp ~(keystone)# vi /etc/neutron/plugins/ml2/ml2_conf.ini
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
root@dlp ~(keystone)# vi /etc/nova/nova.conf
# add follows into [DEFAULT] section
use_neutron = True
linuxnet_interface_driver = nova.network.linux_net.LinuxBridgeInterfaceDriver
firewall_driver = nova.virt.firewall.NoopFirewallDriver
# add follows to the end : Neutron auth info
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
root@dlp ~(keystone)# ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
root@dlp ~(keystone)# su -s /bin/bash neutron -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugin.ini upgrade head"
root@dlp ~(keystone)# systemctl restart neutron-server neutron-metadata-agent nova-api
root@dlp ~(keystone)# systemctl enable neutron-server neutron-metadata-agent
```