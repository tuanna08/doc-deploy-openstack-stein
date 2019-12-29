# (12) Configure Neutron Network

	
Configure Networking for Virtual Machine Instances.
Configure basic settings first for Neutron Services like All in One Settings or Neutron Nodes like here.
For example, configure FLAT type of provider networking on here. The Node has 2 network interfaces like follows.
```
                  eth0|10.0.0.30 
          +-----------+-----------+
          |    [ Control Node ]   |
          |                       |
          |  MariaDB    RabbitMQ  |
          |  Memcached  httpd     |eth1
          |  Keystone   Glance    +----
          |   Nova API,Compute    |
          |    Neutron Server     |
          |  L2,L3,Metadata Agent |
          +-----------------------+
```
[1]	Configure Neutron services.
```
# create a setting file for anonymous interface
# because on Netplan yaml file, anonymous interface never up
# replace the name [eth1] to your environment
root@dlp ~(keystone)# vi /etc/systemd/network/eth1.network
[Match]
Name=eth1

[Network]
LinkLocalAddressing=no
IPv6AcceptRA=no

root@dlp ~(keystone)# systemctl restart systemd-networkd
root@dlp ~(keystone)# vi /etc/neutron/plugins/ml2/ml2_conf.ini
# line 181: add
[ml2_type_flat]
flat_networks = physnet1
root@dlp ~(keystone)# vi /etc/neutron/plugins/ml2/linuxbridge_agent.ini
# line 147: add
[linux_bridge]
physical_interface_mappings = physnet1:eth1
# line 208: uncomment and change
enable_vxlan = false
root@dlp ~(keystone)# systemctl restart neutron-linuxbridge-agent
```
[2]	Create virtual network.
```
root@dlp ~(keystone)# projectID=$(openstack project list | grep service | awk '{print $2}')
# create network named [sharednet1]
root@dlp ~(keystone)# openstack network create --project $projectID \
--share --provider-network-type flat --provider-physical-network physnet1 sharednet1
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2018-06-13T07:10:55Z                 |
| description               |                                      |
| dns_domain                | None                                 |
| id                        | 50855e46-6b0c-48f2-864b-b8c009a4b50f |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| is_default                | False                                |
| is_vlan_transparent       | None                                 |
| mtu                       | 1500                                 |
| name                      | sharednet1                           |
| port_security_enabled     | True                                 |
| project_id                | fa8bf7aca73e44cb9bc2c9f42859857e     |
| provider:network_type     | flat                                 |
| provider:physical_network | physnet1                             |
| provider:segmentation_id  | None                                 |
| qos_policy_id             | None                                 |
| revision_number           | 2                                    |
| router:external           | Internal                             |
| segments                  | None                                 |
| shared                    | True                                 |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tags                      |                                      |
| updated_at                | 2018-06-13T07:10:55Z                 |
+---------------------------+--------------------------------------+

# create subnet [10.0.0.0/24] in [sharednet1]
root@dlp ~(keystone)# openstack subnet create subnet1 --network sharednet1 \
--project $projectID --subnet-range 10.0.0.0/24 \
--allocation-pool start=10.0.0.200,end=10.0.0.254 \
--gateway 10.0.0.1 --dns-nameserver 10.0.0.10
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| allocation_pools  | 10.0.0.200-10.0.0.254                |
| cidr              | 10.0.0.0/24                          |
| created_at        | 2018-06-13T07:11:23Z                 |
| description       |                                      |
| dns_nameservers   | 10.0.0.10                            |
| enable_dhcp       | True                                 |
| gateway_ip        | 10.0.0.1                             |
| host_routes       |                                      |
| id                | 2e848a04-82ec-4eac-80b9-f335875e3b15 |
| ip_version        | 4                                    |
| ipv6_address_mode | None                                 |
| ipv6_ra_mode      | None                                 |
| name              | subnet1                              |
| network_id        | 50855e46-6b0c-48f2-864b-b8c009a4b50f |
| project_id        | fa8bf7aca73e44cb9bc2c9f42859857e     |
| revision_number   | 0                                    |
| segment_id        | None                                 |
| service_types     |                                      |
| subnetpool_id     | None                                 |
| tags              |                                      |
| updated_at        | 2018-06-13T07:11:23Z                 |
+-------------------+--------------------------------------+

# confirm settings
root@dlp ~(keystone)# openstack network list
+--------------------------------------+------------+--------------------------------------+
| ID                                   | Name       | Subnets                              |
+--------------------------------------+------------+--------------------------------------+
| 50855e46-6b0c-48f2-864b-b8c009a4b50f | sharednet1 | 2e848a04-82ec-4eac-80b9-f335875e3b15 |
+--------------------------------------+------------+--------------------------------------+

root@dlp ~(keystone)# openstack subnet list
+--------------------------------------+---------+--------------------------------------+-------------+
| ID                                   | Name    | Network                              | Subnet      |
+--------------------------------------+---------+--------------------------------------+-------------+
| 2e848a04-82ec-4eac-80b9-f335875e3b15 | subnet1 | 50855e46-6b0c-48f2-864b-b8c009a4b50f | 10.0.0.0/24 |
+--------------------------------------+---------+--------------------------------------+-------------+```