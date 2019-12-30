# (18) Configure Neutron (Network Node)

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

[1]	Install some packages for Network Node.
```
root@network:~# apt -y install neutron-plugin-ml2 neutron-plugin-linuxbridge-agent neutron-l3-agent neutron-dhcp-agent neutron-metadata-agent python-neutronclient
```
[2]	Configure as a Network node.
```
root@network:~# mv /etc/neutron/neutron.conf /etc/neutron/neutron.conf.org
root@network:~# vi /etc/neutron/neutron.conf
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

root@network:~# chmod 640 /etc/neutron/neutron.conf
root@network:~# chgrp neutron /etc/neutron/neutron.conf
root@network:~# vi /etc/neutron/l3_agent.ini
# line 17: add
interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
root@network:~# vi /etc/neutron/dhcp_agent.ini
# line 17: add
interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
# line 28: uncomment
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
# line 37: uncomment and change
enable_isolated_metadata = true
root@network:~# vi /etc/neutron/metadata_agent.ini
# line 22: uncomment and specify Nova API server
nova_metadata_host = 10.0.0.30
# line 34: uncomment and specify any secret-words you like
metadata_proxy_shared_secret = metadata_secret
root@network:~# vi /etc/neutron/plugins/ml2/ml2_conf.ini
# line 129: add ( it's OK with no value for [tenant_network_types] (set later if need) )
[ml2]
type_drivers = flat,vlan,vxlan
tenant_network_types =
mechanism_drivers = linuxbridge,l2population
extension_drivers = port_security
# line 262: uncomment and add
enable_security_group = True
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
# last line: uncomment
enable_ipset = True
root@network:~# vi /etc/neutron/plugins/ml2/linuxbridge_agent.ini
# line 235: set own IP address
local_ip = 10.0.0.50
root@network:~# ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
root@network:~# for service in l3-agent dhcp-agent metadata-agent linuxbridge-agent; do
systemctl restart neutron-$service
systemctl enable neutron-$service
done
```