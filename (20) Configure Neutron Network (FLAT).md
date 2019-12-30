# (20) Configure Neutron Network (FLAT)

Configure virtual networking by OpenStack Network Service (Neutron).
For example, configure FLAT type of provider networking on here.
Before it, Configure basic settings on Control Node Network Node Compute Node.
Furthermore, this example is based on the environment that Network Node and Compute Node have 2 network interfaces.
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
     +-----------------------+   +-----------+-----------+   +-----------+-----------+
                                             |eth1                       |eth1

```
[1]	Change settings like follows on both Network Node and Compute Node.
```
root@network:~# vi /etc/neutron/plugins/ml2/ml2_conf.ini
# line 181: add
[ml2_type_flat]
flat_networks = physnet1
root@network:~# vi /etc/neutron/plugins/ml2/linuxbridge_agent.ini
# line 147: add
[linux_bridge]
physical_interface_mappings = physnet1:eth1
# line 208: uncomment and change
enable_vxlan = false
root@network:~# systemctl restart neutron-linuxbridge-agent
```
[2]	Create network. It's OK to work on any node. (This example is on Control Node)
```
root@dlp ~(keystone)# projectID=$(openstack project list | grep service | awk '{print $2}')
# create network named [sharednet1]
root@dlp ~(keystone)# openstack network create --project $projectID \
--share --provider-network-type flat --provider-physical-network physnet1 sharednet1
Created a new network:
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2018-06-15T07:22:26Z                 |
| description               |                                      |
| dns_domain                | None                                 |
| id                        | bfc23d8f-24ce-4a92-97d2-03ad411f2a40 |
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
| updated_at                | 2018-06-15T07:22:26Z                 |
+---------------------------+--------------------------------------+

# create subnet [10.0.0.0/24] in [sharednet1]
root@dlp ~(keystone)# openstack subnet create subnet1 --network sharednet1 \
--project $projectID --subnet-range 10.0.0.0/24 \
--allocation-pool start=10.0.0.200,end=10.0.0.254 \
--gateway 10.0.0.1 --dns-nameserver 10.0.0.10
Created a new subnet:
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| allocation_pools  | 10.0.0.200-10.0.0.254                |
| cidr              | 10.0.0.0/24                          |
| created_at        | 2018-06-15T07:22:52Z                 |
| description       |                                      |
| dns_nameservers   | 10.0.0.10                            |
| enable_dhcp       | True                                 |
| gateway_ip        | 10.0.0.1                             |
| host_routes       |                                      |
| id                | 64253ec7-2ee5-492e-b658-ce3ce4ac9936 |
| ip_version        | 4                                    |
| ipv6_address_mode | None                                 |
| ipv6_ra_mode      | None                                 |
| name              | subnet1                              |
| network_id        | bfc23d8f-24ce-4a92-97d2-03ad411f2a40 |
| project_id        | fa8bf7aca73e44cb9bc2c9f42859857e     |
| revision_number   | 0                                    |
| segment_id        | None                                 |
| service_types     |                                      |
| subnetpool_id     | None                                 |
| tags              |                                      |
| updated_at        | 2018-06-15T07:22:52Z                 |
+-------------------+--------------------------------------+

# confirm settings
root@dlp ~(keystone)# openstack network list
+--------------------------------------+------------+--------------------------------------+
| ID                                   | Name       | Subnets                              |
+--------------------------------------+------------+--------------------------------------+
| bfc23d8f-24ce-4a92-97d2-03ad411f2a40 | sharednet1 | 64253ec7-2ee5-492e-b658-ce3ce4ac9936 |
+--------------------------------------+------------+--------------------------------------+
```
[3]	Create and start a Virtual machine Instance with the network just created above.

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
+--------------------------------------+------------+--------------------------------------+
| ID                                   | Name       | Subnets                              |
+--------------------------------------+------------+--------------------------------------+
| bfc23d8f-24ce-4a92-97d2-03ad411f2a40 | sharednet1 | 64253ec7-2ee5-492e-b658-ce3ce4ac9936 |
+--------------------------------------+------------+--------------------------------------+

# create a security group for instances
ubuntu@dlp ~(keystone)$ openstack security group create secgroup01
+-----------------+-----------------------------------------------------------------------------------+
| Field           | Value                                                                             |
+-----------------+-----------------------------------------------------------------------------------+
| created_at      | 2018-06-15T07:26:14Z                                                              |
| description     | secgroup01                                                                        |
| id              | f1713a90-8389-4d42-a61c-7781fdb2a903                                              |
| name            | secgroup01                                                                        |
| project_id      | 9b4516711a0a4543ba5adfe7510ea1a1                                                  |
| revision_number | 2                                                                                 |
| rules           | created_at='2018-06-15T07:26:14Z', direction='egress', ethertype='IPv4', id='6... |
|                 | created_at='2018-06-15T07:26:14Z', direction='egress', ethertype='IPv6', id='e... |
| updated_at      | 2018-06-15T07:26:14Z                                                              |
+-----------------+-----------------------------------------------------------------------------------+

# create a SSH keypair for connecting to instances
ubuntu@dlp ~(keystone)$ ssh-keygen -q -N ""
Enter file in which to save the key (/home/ubuntu/.ssh/id_rsa):
# add public-key
ubuntu@dlp ~(keystone)$ openstack keypair create --public-key ~/.ssh/id_rsa.pub mykey
+-------------+-------------------------------------------------+
| Field       | Value                                           |
+-------------+-------------------------------------------------+
| fingerprint | f5:17:31:c3:eb:90:dc:d5:e4:50:41:47:8c:cb:55:3b |
| name        | mykey                                           |
| user_id     | a212f017161d42228c47b3aef27f9f6c                |
+-------------+-------------------------------------------------+

ubuntu@dlp ~(keystone)$ netID=$(openstack network list | grep sharednet1 | awk '{ print $2 }')
ubuntu@dlp ~(keystone)$ openstack server create --flavor m1.small --image Ubuntu1804 --security-group secgroup01 --nic net-id=$netID --key-name mykey Ubuntu_1804
ubuntu@dlp ~(keystone)$ openstack server list
+--------------------------------------+-------------+--------+-----------------------+------------+----------+
| ID                                   | Name        | Status | Networks              | Image      | Flavor   |
+--------------------------------------+-------------+--------+-----------------------+------------+----------+
| 833834ef-de72-4a48-90d8-25ac6beacc9f | Ubuntu_1804 | ACTIVE | sharednet1=10.0.0.208 | Ubuntu1804 | m1.small |
+--------------------------------------+-------------+--------+-----------------------+------------+----------+
```
[4]	Configure security settings for the security group you created above to access with SSH and ICMP.
```
# permit ICMP
ubuntu@dlp ~(keystone)$ openstack security group rule create --protocol icmp --ingress default
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| created_at        | 2018-06-15T07:41:06Z                 |
| description       |                                      |
| direction         | ingress                              |
| ether_type        | IPv4                                 |
| id                | c949ae40-65b4-4be6-a375-054749815b7d |
| name              | None                                 |
| port_range_max    | None                                 |
| port_range_min    | None                                 |
| project_id        | 9b4516711a0a4543ba5adfe7510ea1a1     |
| protocol          | icmp                                 |
| remote_group_id   | None                                 |
| remote_ip_prefix  | 0.0.0.0/0                            |
| revision_number   | 0                                    |
| security_group_id | f5ddf9ce-9c8d-4c78-95ea-1099e294274c |
| updated_at        | 2018-06-15T07:41:06Z                 |
+-------------------+--------------------------------------+

# permit SSH
ubuntu@dlp ~(keystone)$ openstack security group rule create --protocol tcp --dst-port 22:22 default
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| created_at        | 2018-06-15T07:41:19Z                 |
| description       |                                      |
| direction         | ingress                              |
| ether_type        | IPv4                                 |
| id                | 4a75366e-cacc-49b6-b2bd-8fe94873778c |
| name              | None                                 |
| port_range_max    | 22                                   |
| port_range_min    | 22                                   |
| project_id        | 9b4516711a0a4543ba5adfe7510ea1a1     |
| protocol          | tcp                                  |
| remote_group_id   | None                                 |
| remote_ip_prefix  | 0.0.0.0/0                            |
| revision_number   | 0                                    |
| security_group_id | f5ddf9ce-9c8d-4c78-95ea-1099e294274c |
| updated_at        | 2018-06-15T07:41:19Z                 |
+-------------------+--------------------------------------+

ubuntu@dlp ~(keystone)$ openstack security group rule list
+----------+-------------+-----------+------------+--------------------------------------+-------------------+
| ID       | IP Protocol | IP Range  | Port Range | Remote Security Group                | Security Group    |
+----------+-------------+-----------+------------+--------------------------------------+-------------------+
| 2792d... | None        | None      |            | f5ddf9ce-9c8d-4c78-95ea-1099e294274c | f5ddf9ce-9c8d-... |
| 3303e... | None        | None      |            | None                                 | f5ddf9ce-9c8d-... |
| 4a753... | tcp         | 0.0.0.0/0 | 22:22      | None                                 | f5ddf9ce-9c8d-... |
| 6b888... | None        | None      |            | None                                 | f1713a90-8389-... |
| b14b0... | None        | None      |            | f5ddf9ce-9c8d-4c78-95ea-1099e294274c | f5ddf9ce-9c8d-... |
| b6051... | None        | None      |            | None                                 | f5ddf9ce-9c8d-... |
| c949a... | icmp        | 0.0.0.0/0 |            | None                                 | f5ddf9ce-9c8d-... |
| e8077... | None        | None      |            | None                                 | f1713a90-8389-... |
+----------+-------------+-----------+------------+--------------------------------------+-------------------+
```
[5]	Login to Instance.
```
ubuntu@dlp ~(keystone)$ ssh ubuntu@10.0.0.208
The authenticity of host '10.0.0.208 (10.0.0.208)' can't be established.
ECDSA key fingerprint is SHA256:/23tlmEk91VOrjfJwpxJLcJo1KSaSHoO9VuAu1uFrgw.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.0.0.208' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 18.04 LTS (GNU/Linux 4.15.0-23-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri Jun 15 16:43:04 JST 2018

  System load:  0.14              Processes:           89
  Usage of /:   19.5% of 8.80GB   Users logged in:     0
  Memory usage: 6%                IP address for ens3: 10.0.0.208
  Swap usage:   0%

23 packages can be updated.
18 updates are security updates.

ubuntu@ubuntu-1804:~$     # just logined
```