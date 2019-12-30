#(21) Configure Neutron Network (VXLAN)

Configure virtual networking by OpenStack Network Service (Neutron).
For example, configure VXLAN type of networking on here.
Before it, Configure basic settings on Control Node Network Node Compute Node.
Furthermore, this example is based on the environment that Network Node has 2 network interfaces.
And also eth1 is up without IP Address, refer to here of [1] to up anonymous interface on Netplan.

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
     +-----------------------+   +-----------+-----------+   +-----------------------+
                                             |eth1
```
[1]	Change settings on Control Node.
```
root@dlp ~(keystone)# vi /etc/neutron/plugins/ml2/ml2_conf.ini
# line 130: add a value to tenant_network_types
tenant_network_types = vxlan
# line 181: add
[ml2_type_flat]
flat_networks = physnet1
# line 235: add
[ml2_type_vxlan]
vni_ranges = 1:1000
root@dlp ~(keystone)# systemctl restart neutron-server
```
[2]	Change settings on Network Node.
```
root@network:~# vi /etc/neutron/plugins/ml2/ml2_conf.ini
# line 130: add a value to tenant_network_types
tenant_network_types = vxlan
# line 181: add
[ml2_type_flat]
flat_networks = physnet1
# line 235: add
[ml2_type_vxlan]
vni_ranges = 1:1000
root@network:~# vi /etc/neutron/plugins/ml2/linuxbridge_agent.ini
# line 118: add
[agent]
prevent_arp_spoofing = True
# line 147: add
[linux_bridge]
physical_interface_mappings = physnet1:eth1
# line 201: add
[vxlan]
enable_vxlan = True
l2_population = True
root@network:~# vi /etc/neutron/dhcp_agent.ini
# line 63: add
dnsmasq_config_file = /etc/neutron/dnsmasq-neutron.conf
root@network:~# vi /etc/neutron/dnsmasq-neutron.conf
# create new
dhcp-option-force=26,1450
root@network:~# for service in l3-agent dhcp-agent metadata-agent linuxbridge-agent; do
systemctl restart neutron-$service
done
```
[3]	Change settings on Compute Node.
```
root@node01:~# vi /etc/neutron/plugins/ml2/ml2_conf.ini
# line 130: add a value to tenant_network_types
tenant_network_types = vxlan
# line 181: add
[ml2_type_flat]
flat_networks = physnet1
# line 235: add
[ml2_type_vxlan]
vni_ranges = 1:1000
root@node01:~# vi /etc/neutron/plugins/ml2/linuxbridge_agent.ini
# line 118: add
[agent]
prevent_arp_spoofing = True
# line 201: add
[vxlan]
enable_vxlan = True
l2_population = True
root@node01:~# systemctl restart neutron-linuxbridge-agent
```
[4]	Create a Virtual router. It's OK to work on any node. (This example is on Control Node)
```
root@dlp ~(keystone)# openstack router create router01
+-------------------------+--------------------------------------+
| Field                   | Value                                |
+-------------------------+--------------------------------------+
| admin_state_up          | UP                                   |
| availability_zone_hints |                                      |
| availability_zones      |                                      |
| created_at              | 2018-06-15T07:56:21Z                 |
| description             |                                      |
| distributed             | False                                |
| external_gateway_info   | None                                 |
| flavor_id               | None                                 |
| ha                      | False                                |
| id                      | 67ffb912-6afd-4c5f-8980-54d76abf7e1a |
| name                    | router01                             |
| project_id              | 12d80218a35e4c62aa0403e92b5b649a     |
| revision_number         | 0                                    |
| routes                  |                                      |
| status                  | ACTIVE                               |
| tags                    |                                      |
| updated_at              | 2018-06-15T07:56:21Z                 |
+-------------------------+--------------------------------------+
```
[5]	Create internal network and associate with the router above.
```
# create internal network
root@dlp ~(keystone)# openstack network create int_net --provider-network-type vxlan
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2018-06-15T07:56:37Z                 |
| description               |                                      |
| dns_domain                | None                                 |
| id                        | 04a14e8b-5ad8-4db7-b89c-cef830b487af |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| is_default                | False                                |
| is_vlan_transparent       | None                                 |
| mtu                       | 1450                                 |
| name                      | int_net                              |
| port_security_enabled     | True                                 |
| project_id                | 12d80218a35e4c62aa0403e92b5b649a     |
| provider:network_type     | vxlan                                |
| provider:physical_network | None                                 |
| provider:segmentation_id  | 31                                   |
| qos_policy_id             | None                                 |
| revision_number           | 2                                    |
| router:external           | Internal                             |
| segments                  | None                                 |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tags                      |                                      |
| updated_at                | 2018-06-15T07:56:38Z                 |
+---------------------------+--------------------------------------+

# create subnet in the internal network
root@dlp ~(keystone)# openstack subnet create subnet1 --network int_net \
--subnet-range 192.168.100.0/24 --gateway 192.168.100.1 \
--dns-nameserver 10.0.0.10
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| allocation_pools  | 192.168.100.2-192.168.100.254        |
| cidr              | 192.168.100.0/24                     |
| created_at        | 2018-06-15T07:56:59Z                 |
| description       |                                      |
| dns_nameservers   | 10.0.0.10                            |
| enable_dhcp       | True                                 |
| gateway_ip        | 192.168.100.1                        |
| host_routes       |                                      |
| id                | cccfe09e-6ed6-4a11-8aa5-5e56abe4e181 |
| ip_version        | 4                                    |
| ipv6_address_mode | None                                 |
| ipv6_ra_mode      | None                                 |
| name              | subnet1                              |
| network_id        | 04a14e8b-5ad8-4db7-b89c-cef830b487af |
| project_id        | 12d80218a35e4c62aa0403e92b5b649a     |
| revision_number   | 0                                    |
| segment_id        | None                                 |
| service_types     |                                      |
| subnetpool_id     | None                                 |
| tags              |                                      |
| updated_at        | 2018-06-15T07:56:59Z                 |
+-------------------+--------------------------------------+

# set internal network to the router above
root@dlp ~(keystone)# openstack router add subnet router01 subnet1
```
[6]	Create external network and associate with the router above.
```
# create external network
root@dlp ~(keystone)# openstack network create \
--provider-physical-network physnet1 \
--provider-network-type flat --external ext_net
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2018-06-15T07:57:31Z                 |
| description               |                                      |
| dns_domain                | None                                 |
| id                        | 38f24fdf-b038-4e50-8923-a3e05f8683d0 |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| is_default                | False                                |
| is_vlan_transparent       | None                                 |
| mtu                       | 1500                                 |
| name                      | ext_net                              |
| port_security_enabled     | True                                 |
| project_id                | 12d80218a35e4c62aa0403e92b5b649a     |
| provider:network_type     | flat                                 |
| provider:physical_network | physnet1                             |
| provider:segmentation_id  | None                                 |
| qos_policy_id             | None                                 |
| revision_number           | 5                                    |
| router:external           | External                             |
| segments                  | None                                 |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tags                      |                                      |
| updated_at                | 2018-06-15T07:57:31Z                 |
+---------------------------+--------------------------------------+

# create subnet in external network
root@dlp ~(keystone)# openstack subnet create subnet2 \
--network ext_net --subnet-range 10.0.0.0/24 \
--allocation-pool start=10.0.0.200,end=10.0.0.254 \
--gateway 10.0.0.1 --dns-nameserver 10.0.0.10 --no-dhcp
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| allocation_pools  | 10.0.0.200-10.0.0.254                |
| cidr              | 10.0.0.0/24                          |
| created_at        | 2018-06-15T07:57:51Z                 |
| description       |                                      |
| dns_nameservers   | 10.0.0.10                            |
| enable_dhcp       | False                                |
| gateway_ip        | 10.0.0.1                             |
| host_routes       |                                      |
| id                | ad133394-8c9f-4cd0-9db8-638f78e6f29b |
| ip_version        | 4                                    |
| ipv6_address_mode | None                                 |
| ipv6_ra_mode      | None                                 |
| name              | subnet2                              |
| network_id        | 38f24fdf-b038-4e50-8923-a3e05f8683d0 |
| project_id        | 12d80218a35e4c62aa0403e92b5b649a     |
| revision_number   | 0                                    |
| segment_id        | None                                 |
| service_types     |                                      |
| subnetpool_id     | None                                 |
| tags              |                                      |
| updated_at        | 2018-06-15T07:57:51Z                 |
+-------------------+--------------------------------------+

# set gateway to the router above
root@dlp ~(keystone)# openstack router set router01 --external-gateway ext_net
```
[7]	By default, it's possible to access for all projects to external network, but for internal network, only admin projects can access to it, so grant access permission of internal network to a project you'd like to let users in the project use.
```
# show network RBAC list
root@dlp ~(keystone)# openstack network rbac list
+--------------------------------------+-------------+--------------------------------------+
| ID                                   | Object Type | Object ID                            |
+--------------------------------------+-------------+--------------------------------------+
| 719d4c19-0612-4348-a970-afc7e456c26f | network     | 38f24fdf-b038-4e50-8923-a3e05f8683d0 |
+--------------------------------------+-------------+--------------------------------------+

# RBAC details (all projects can access only for access_as_external)
root@dlp ~(keystone)# openstack network rbac show 719d4c19-0612-4348-a970-afc7e456c26f
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| action            | access_as_external                   |
| id                | 719d4c19-0612-4348-a970-afc7e456c26f |
| name              | None                                 |
| object_id         | 38f24fdf-b038-4e50-8923-a3e05f8683d0 |
| object_type       | network                              |
| project_id        | 12d80218a35e4c62aa0403e92b5b649a     |
| target_project_id | *                                    |
+-------------------+--------------------------------------+

# show network list
root@dlp ~(keystone)# openstack network list
+--------------------------------------+---------+--------------------------------------+
| ID                                   | Name    | Subnets                              |
+--------------------------------------+---------+--------------------------------------+
| 04a14e8b-5ad8-4db7-b89c-cef830b487af | int_net | cccfe09e-6ed6-4a11-8aa5-5e56abe4e181 |
| 38f24fdf-b038-4e50-8923-a3e05f8683d0 | ext_net | ad133394-8c9f-4cd0-9db8-638f78e6f29b |
+--------------------------------------+---------+--------------------------------------+

# show project list
root@dlp ~(keystone)# openstack project list
+----------------------------------+---------+
| ID                               | Name    |
+----------------------------------+---------+
| 12d80218a35e4c62aa0403e92b5b649a | admin   |
| fa8bf7aca73e44cb9bc2c9f42859857e | service |
+----------------------------------+---------+

# grant [access_as_shared] permission for [int_net] to [hiroshima] project
root@dlp ~(keystone)# netID=$(openstack network list | grep int_net | awk '{ print $2 }')
root@dlp ~(keystone)# prjID=$(openstack project list | grep hiroshima | awk '{ print $2 }')
root@dlp ~(keystone)# openstack network rbac create --target-project $prjID --type network --action access_as_shared $netID
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| action            | access_as_shared                     |
| id                | ec830892-f7e3-43ba-9c1f-6fca715658ce |
| name              | None                                 |
| object_id         | 04a14e8b-5ad8-4db7-b89c-cef830b487af |
| object_type       | network                              |
| project_id        | 12d80218a35e4c62aa0403e92b5b649a     |
| target_project_id | 04a16d601dc940dd845f3092ce2712e8     |
+-------------------+--------------------------------------+
```
[8]	Login with a user who is in the project you granted access permission to internal network and Create and boot an instance.
```
# show flavor list
ubuntu@dlp ~(keystone)$ openstack flavor list
+----+----------+------+------+-----------+-------+-----------+
| ID | Name     |  RAM | Disk | Ephemeral | VCPUs | Is Public |
+----+----------+------+------+-----------+-------+-----------+
| 0  | m1.small | 2048 |   10 |         0 |     1 | True      |
+----+----------+------+------+-----------+-------+-----------+

# show image list
ubuntu@dlp ~(keystone)$ openstack image list
+--------------------------------------+------------+--------+
| ID                                   | Name       | Status |
+--------------------------------------+------------+--------+
| bb285a87-fe42-449a-a659-b5a304177b18 | Ubuntu1804 | active |
+--------------------------------------+------------+--------+

# show network list
ubuntu@dlp ~(keystone)$ openstack network list
+--------------------------------------+---------+--------------------------------------+
| ID                                   | Name    | Subnets                              |
+--------------------------------------+---------+--------------------------------------+
| 04a14e8b-5ad8-4db7-b89c-cef830b487af | int_net | cccfe09e-6ed6-4a11-8aa5-5e56abe4e181 |
| 38f24fdf-b038-4e50-8923-a3e05f8683d0 | ext_net | ad133394-8c9f-4cd0-9db8-638f78e6f29b |
+--------------------------------------+---------+--------------------------------------+

# create a security group for instances
ubuntu@dlp ~(keystone)$ openstack security group create secgroup01
+-----------------+----------------------------------------------------------------------------------+
| Field           | Value                                                                            |
+-----------------+----------------------------------------------------------------------------------+
| created_at      | 2018-06-15T08:03:34Z                                                             |
| description     | secgroup01                                                                       |
| id              | 9c482015-3864-47e6-8a91-93d36c6f47f3                                             |
| name            | secgroup01                                                                       |
| project_id      | 04a16d601dc940dd845f3092ce2712e8                                                 |
| revision_number | 2                                                                                |
| rules           | created_at='2018-06-15T08:03:34Z', direction='egress', ethertype='IPv4', id='... |
|                 | created_at='2018-06-15T08:03:34Z', direction='egress', ethertype='IPv6', id='... |
| updated_at      | 2018-06-15T08:03:34Z                                                             |
+-----------------+----------------------------------------------------------------------------------+

# create a SSH keypair for connecting to instances
ubuntu@dlp ~(keystone)$ ssh-keygen -q -N ""
Enter file in which to save the key (/home/ubuntu/.ssh/id_rsa):
# add public-key
ubuntu@dlp ~(keystone)$ openstack keypair create --public-key ~/.ssh/id_rsa.pub mykey
+-------------+-------------------------------------------------+
| Field       | Value                                           |
+-------------+-------------------------------------------------+
| fingerprint | 56:07:9c:50:df:17:68:7d:d3:9d:69:ab:e6:bf:4a:c8 |
| name        | mykey                                           |
| user_id     | 601473fae2c444a484860046ef2484e2                |
+-------------+-------------------------------------------------+

ubuntu@dlp ~(keystone)$ netID=$(openstack network list | grep int_net | awk '{ print $2 }')
ubuntu@dlp ~(keystone)$ openstack server create --flavor m1.small --image Ubuntu1804 --security-group secgroup01 --nic net-id=$netID --key-name mykey Ubuntu_1804
ubuntu@dlp ~(keystone)$ openstack server list
+--------------------------------------+-------------+--------+-----------------------+------------+----------+
| ID                                   | Name        | Status | Networks              | Image      | Flavor   |
+--------------------------------------+-------------+--------+-----------------------+------------+----------+
| 3aae02ab-b9b7-4edf-81e2-fea2df72fc8e | Ubuntu_1804 | ACTIVE | int_net=192.168.100.5 | Ubuntu1804 | m1.small |
+--------------------------------------+-------------+--------+-----------------------+------------+----------+
```
[9]	Assign floating IP address to the Instance above.
```
ubuntu@dlp ~(keystone)$ openstack floating ip create ext_net
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| created_at          | 2018-06-15T08:06:17Z                 |
| description         |                                      |
| fixed_ip_address    | None                                 |
| floating_ip_address | 10.0.0.205                           |
| floating_network_id | 38f24fdf-b038-4e50-8923-a3e05f8683d0 |
| id                  | f5eaf19a-e6f3-4078-b4c5-2bbdacb99260 |
| name                | 10.0.0.205                           |
| port_id             | None                                 |
| project_id          | 04a16d601dc940dd845f3092ce2712e8     |
| qos_policy_id       | None                                 |
| revision_number     | 0                                    |
| router_id           | None                                 |
| status              | DOWN                                 |
| subnet_id           | None                                 |
| updated_at          | 2018-06-15T08:06:17Z                 |
+---------------------+--------------------------------------+

ubuntu@dlp ~(keystone)$ openstack server add floating ip Ubuntu_1804 10.0.0.205
# confirm settings
ubuntu@dlp ~(keystone)$ openstack floating ip show 10.0.0.205
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| created_at          | 2018-06-15T08:06:17Z                 |
| description         |                                      |
| fixed_ip_address    | 192.168.100.5                        |
| floating_ip_address | 10.0.0.205                           |
| floating_network_id | 38f24fdf-b038-4e50-8923-a3e05f8683d0 |
| id                  | f5eaf19a-e6f3-4078-b4c5-2bbdacb99260 |
| name                | 10.0.0.205                           |
| port_id             | fcf7b6a1-2cb4-4485-bd66-edab8384bb85 |
| project_id          | 04a16d601dc940dd845f3092ce2712e8     |
| qos_policy_id       | None                                 |
| revision_number     | 2                                    |
| router_id           | 67ffb912-6afd-4c5f-8980-54d76abf7e1a |
| status              | ACTIVE                               |
| subnet_id           | None                                 |
| updated_at          | 2018-06-15T08:06:53Z                 |
+---------------------+--------------------------------------+

ubuntu@dlp ~(keystone)$ openstack server list
+--------------+-------------+--------+-----------------------------------+------------+----------+
| ID           | Name        | Status | Networks                          | Image      | Flavor   |
+--------------+-------------+--------+-----------------------------------+------------+----------+
| 3aae02ab-... | Ubuntu_1804 | ACTIVE | int_net=192.168.100.5, 10.0.0.205 | Ubuntu1804 | m1.small |
+--------------+-------------+--------+-----------------------------------+------------+----------+
```
[10]	Configure security settings for the security group you created above to access with SSH and ICMP.
# permit ICMP
```
ubuntu@dlp ~(keystone)$ openstack security group rule create --protocol icmp --ingress default
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| created_at        | 2018-06-15T08:07:35Z                 |
| description       |                                      |
| direction         | ingress                              |
| ether_type        | IPv4                                 |
| id                | 838a755e-aa5c-4724-9b86-33358487828c |
| name              | None                                 |
| port_range_max    | None                                 |
| port_range_min    | None                                 |
| project_id        | 04a16d601dc940dd845f3092ce2712e8     |
| protocol          | icmp                                 |
| remote_group_id   | None                                 |
| remote_ip_prefix  | 0.0.0.0/0                            |
| revision_number   | 0                                    |
| security_group_id | ed935a79-0923-48cc-ab87-f8b522da4fef |
| updated_at        | 2018-06-15T08:07:35Z                 |
+-------------------+--------------------------------------+

# permit SSH
ubuntu@dlp ~(keystone)$ openstack security group rule create --protocol tcp --dst-port 22:22 default
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| created_at        | 2018-06-15T08:07:48Z                 |
| description       |                                      |
| direction         | ingress                              |
| ether_type        | IPv4                                 |
| id                | f9a597b7-a75a-4b69-a5aa-6c8d19b19993 |
| name              | None                                 |
| port_range_max    | 22                                   |
| port_range_min    | 22                                   |
| project_id        | 04a16d601dc940dd845f3092ce2712e8     |
| protocol          | tcp                                  |
| remote_group_id   | None                                 |
| remote_ip_prefix  | 0.0.0.0/0                            |
| revision_number   | 0                                    |
| security_group_id | ed935a79-0923-48cc-ab87-f8b522da4fef |
| updated_at        | 2018-06-15T08:07:48Z                 |
+-------------------+--------------------------------------+

ubuntu@dlp ~(keystone)$ openstack security group rule list
+----------+-------------+-----------+------------+--------------------------------------+-------------------+
| ID       | IP Protocol | IP Range  | Port Range | Remote Security Group                | Security Group    |
+----------+-------------+-----------+------------+--------------------------------------+-------------------+
| 1487e... | None        | None      |            | None                                 | 9c482015-3864-... |
| 69d3d... | None        | None      |            | ed935a79-0923-48cc-ab87-f8b522da4fef | ed935a79-0923-... |
| 838a7... | icmp        | 0.0.0.0/0 |            | None                                 | ed935a79-0923-... |
| b5b0a... | None        | None      |            | None                                 | ed935a79-0923-... |
| dff54... | None        | None      |            | None                                 | 9c482015-3864-... |
| ecb34... | None        | None      |            | ed935a79-0923-48cc-ab87-f8b522da4fef | ed935a79-0923-... |
| f9a59... | tcp         | 0.0.0.0/0 | 22:22      | None                                 | ed935a79-0923-... |
| fae23... | None        | None      |            | None                                 | ed935a79-0923-... |
+----------+-------------+-----------+------------+--------------------------------------+-------------------+
```
[11]	It's possible to login to the Instance to connect to the floating IP address with SSH like follows.
```
ubuntu@dlp ~(keystone)$ openstack server list
+--------------+-------------+--------+-----------------------------------+------------+----------+
| ID           | Name        | Status | Networks                          | Image      | Flavor   |
+--------------+-------------+--------+-----------------------------------+------------+----------+
| 3aae02ab-... | Ubuntu_1804 | ACTIVE | int_net=192.168.100.5, 10.0.0.205 | Ubuntu1804 | m1.small |
+--------------+-------------+--------+-----------------------------------+------------+----------+

ubuntu@dlp ~(keystone)$ ssh ubuntu@10.0.0.205
The authenticity of host '10.0.0.205 (10.0.0.205)' can't be established.
ECDSA key fingerprint is SHA256:qtOi0KLZ6uAllbP1AytZfQ43ZTl4JqgRwRW/pvWoi4g.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.0.0.205' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 18.04 LTS (GNU/Linux 4.15.0-23-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri Jun 15 17:10:01 JST 2018

  System load:  0.0               Processes:           89
  Usage of /:   19.5% of 8.80GB   Users logged in:     0
  Memory usage: 6%                IP address for ens3: 192.168.100.5
  Swap usage:   0%

23 packages can be updated.
18 updates are security updates.

ubuntu@ubuntu-1804:~$     # just logined
```