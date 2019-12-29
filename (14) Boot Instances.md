# (14) Boot Instances

Create and Start Virtual Machine Instance.
[1]	Login with a user and create a config for authentication of Keystyone. The username or password in the config are just the one you added in keystone like here. Next Create and run an instance.
```
ubuntu@dlp:~$ vi ~/keystonerc
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=hiroshima
export OS_USERNAME=serverworld
export OS_PASSWORD=userpassword
export OS_AUTH_URL=http://10.0.0.30:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
export PS1='\u@\h \W(keystone)\$ '
ubuntu@dlp:~$ chmod 600 ~/keystonerc
ubuntu@dlp:~$ source ~/keystonerc
ubuntu@dlp ~(keystone)$ echo "source ~/keystonerc " >> ~/.bash_profile
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
| 50855e46-6b0c-48f2-864b-b8c009a4b50f | sharednet1 | 2e848a04-82ec-4eac-80b9-f335875e3b15 |
+--------------------------------------+------------+--------------------------------------+

# create a security group for instances
ubuntu@dlp ~(keystone)$ openstack security group create secgroup01
+-----------------+-------------------------------------------------------------------------+
| Field           | Value                                                                   |
+-----------------+-------------------------------------------------------------------------+
| created_at      | 2018-06-13T07:16:04Z                                                    |
| description     | secgroup01                                                              |
| id              | e44eaba8-cf7f-4286-a0e5-e944dfe034cd                                    |
| name            | secgroup01                                                              |
| project_id      | 78d261e9ec6c484b927a6006e780306c                                        |
| revision_number | 2                                                                       |
| rules           | created_at='2018-06-13T07:16:04Z', direction='egress', ethertype='I ... |
|                 | created_at='2018-06-13T07:16:04Z', direction='egress', ethertype='I ... |
| updated_at      | 2018-06-13T07:16:04Z                                                    |
+-----------------+-------------------------------------------------------------------------+

ubuntu@dlp ~(keystone)$ openstack security group list
+--------------+------------+------------------------+----------------------------------+
| ID           | Name       | Description            | Project                          |
+--------------+------------+------------------------+----------------------------------+
| 1423e27c-... | default    | Default security group | 78d261e9ec6c484b927a6006e780306c |
| e44eaba8-... | secgroup01 | secgroup01             | 78d261e9ec6c484b927a6006e780306c |
+--------------+------------+------------------------+----------------------------------+

# create a SSH keypair for connecting to instances
ubuntu@dlp ~(keystone)$ ssh-keygen -q -N ""
Enter file in which to save the key (/home/ubuntu/.ssh/id_rsa):
# add public-key
ubuntu@dlp ~(keystone)$ openstack keypair create --public-key ~/.ssh/id_rsa.pub mykey
+-------------+-------------------------------------------------+
| Field       | Value                                           |
+-------------+-------------------------------------------------+
| fingerprint | 1f:13:f5:72:f6:91:99:2a:fa:e9:dd:f9:94:aa:c0:b7 |
| name        | mykey                                           |
| user_id     | 7992021fa3e34c9bbe2d86f5beebd01c                |
+-------------+-------------------------------------------------+

ubuntu@dlp ~(keystone)$ openstack keypair list
+-------+-------------------------------------------------+
| Name  | Fingerprint                                     |
+-------+-------------------------------------------------+
| mykey | 1f:13:f5:72:f6:91:99:2a:fa:e9:dd:f9:94:aa:c0:b7 |
+-------+-------------------------------------------------+

ubuntu@dlp ~(keystone)$ netID=$(openstack network list | grep sharednet1 | awk '{ print $2 }')
# create and boot an instance
ubuntu@dlp ~(keystone)$ openstack server create --flavor m1.small --image Ubuntu1804 --security-group secgroup01 --nic net-id=$netID --key-name mykey Ubuntu_1804
+-----------------------------+---------------------------------------------------+
| Field                       | Value                                             |
+-----------------------------+---------------------------------------------------+
| OS-DCF:diskConfig           | MANUAL                                            |
| OS-EXT-AZ:availability_zone |                                                   |
| OS-EXT-STS:power_state      | NOSTATE                                           |
| OS-EXT-STS:task_state       | scheduling                                        |
| OS-EXT-STS:vm_state         | building                                          |
| OS-SRV-USG:launched_at      | None                                              |
| OS-SRV-USG:terminated_at    | None                                              |
| accessIPv4                  |                                                   |
| accessIPv6                  |                                                   |
| addresses                   |                                                   |
| adminPass                   | 8BdRh6YfvU2X                                      |
| config_drive                |                                                   |
| created                     | 2018-06-13T07:19:13Z                              |
| flavor                      | m1.small (0)                                      |
| hostId                      |                                                   |
| id                          | deea0234-8e16-4f9c-8622-34a81bf03b1a              |
| image                       | Ubuntu1804 (bb285a87-fe42-449a-a659-b5a304177b18) |
| key_name                    | mykey                                             |
| name                        | Ubuntu_1804                                       |
| progress                    | 0                                                 |
| project_id                  | 78d261e9ec6c484b927a6006e780306c                  |
| properties                  |                                                   |
| security_groups             | name='e44eaba8-cf7f-4286-a0e5-e944dfe034cd'       |
| status                      | BUILD                                             |
| updated                     | 2018-06-13T07:19:13Z                              |
| user_id                     | 7992021fa3e34c9bbe2d86f5beebd01c                  |
| volumes_attached            |                                                   |
+-----------------------------+---------------------------------------------------+

# show status ([BUILD] status is shown when building instance)
ubuntu@dlp ~(keystone)$ openstack server list
+--------------------------------------+-------------+--------+----------+------------+----------+
| ID                                   | Name        | Status | Networks | Image      | Flavor   |
+--------------------------------------+-------------+--------+----------+------------+----------+
| deea0234-8e16-4f9c-8622-34a81bf03b1a | Ubuntu_1804 | BUILD  |          | Ubuntu1804 | m1.small |
+--------------------------------------+-------------+--------+----------+------------+----------+

# when starting noramlly, the status turns to [ACTIVE]
ubuntu@dlp ~(keystone)$ openstack server list
+--------------------------------------+-------------+--------+-----------------------+------------+----------+
| ID                                   | Name        | Status | Networks              | Image      | Flavor   |
+--------------------------------------+-------------+--------+-----------------------+------------+----------+
| deea0234-8e16-4f9c-8622-34a81bf03b1a | Ubuntu_1804 | ACTIVE | sharednet1=10.0.0.202 | Ubuntu1804 | m1.small |
+--------------------------------------+-------------+--------+-----------------------+------------+----------+
```
[2]	Configure security settings for the security group you created above to access with SSH and ICMP.
```
# permit ICMP
ubuntu@dlp ~(keystone)$ openstack security group rule create --protocol icmp --ingress secgroup01
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| created_at        | 2018-06-13T07:21:27Z                 |
| description       |                                      |
| direction         | ingress                              |
| ether_type        | IPv4                                 |
| id                | f8af617c-0d3b-49e2-86ad-a3f5d8578437 |
| name              | None                                 |
| port_range_max    | None                                 |
| port_range_min    | None                                 |
| project_id        | 78d261e9ec6c484b927a6006e780306c     |
| protocol          | icmp                                 |
| remote_group_id   | None                                 |
| remote_ip_prefix  | 0.0.0.0/0                            |
| revision_number   | 0                                    |
| security_group_id | e44eaba8-cf7f-4286-a0e5-e944dfe034cd |
| updated_at        | 2018-06-13T07:21:27Z                 |
+-------------------+--------------------------------------+

# permit SSH
ubuntu@dlp ~(keystone)$ openstack security group rule create --protocol tcp --dst-port 22:22 secgroup01
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| created_at        | 2018-06-13T07:21:41Z                 |
| description       |                                      |
| direction         | ingress                              |
| ether_type        | IPv4                                 |
| id                | 99cdab23-b7e5-4552-afe1-4bf2422d5e2d |
| name              | None                                 |
| port_range_max    | 22                                   |
| port_range_min    | 22                                   |
| project_id        | 78d261e9ec6c484b927a6006e780306c     |
| protocol          | tcp                                  |
| remote_group_id   | None                                 |
| remote_ip_prefix  | 0.0.0.0/0                            |
| revision_number   | 0                                    |
| security_group_id | e44eaba8-cf7f-4286-a0e5-e944dfe034cd |
| updated_at        | 2018-06-13T07:21:41Z                 |
+-------------------+--------------------------------------+

ubuntu@dlp ~(keystone)$ openstack security group rule list
+--------------+-------------+-----------+------------+--------------------------------------+-------------------+
| ID           | IP Protocol | IP Range  | Port Range | Remote Security Group                | Security Group    |
+--------------+-------------+-----------+------------+--------------------------------------+-------------------+
| 10bf4c0c-... | None        | None      |            | None                                 | 1423e27c-f7c8-... |
| 2c9432d0-... | None        | None      |            | 1423e27c-f7c8-462f-8da8-35d04990377c | 1423e27c-f7c8-... |
| 39e75a2b-... | None        | None      |            | None                                 | e44eaba8-cf7f-... |
| 7371a399-... | None        | None      |            | 1423e27c-f7c8-462f-8da8-35d04990377c | 1423e27c-f7c8-... |
| 99cdab23-... | tcp         | 0.0.0.0/0 | 22:22      | None                                 | e44eaba8-cf7f-... |
| abc0a4a2-... | None        | None      |            | None                                 | 1423e27c-f7c8-... |
| d5160d34-... | None        | None      |            | None                                 | e44eaba8-cf7f-... |
| f8af617c-... | icmp        | 0.0.0.0/0 |            | None                                 | e44eaba8-cf7f-... |
+--------------+-------------+-----------+------------+--------------------------------------+-------------------+```
[3]	Login to the instance with SSH.
```
ubuntu@dlp ~(keystone)$ openstack server list
+--------------------------------------+-------------+--------+-----------------------+------------+----------+
| ID                                   | Name        | Status | Networks              | Image      | Flavor   |
+--------------------------------------+-------------+--------+-----------------------+------------+----------+
| deea0234-8e16-4f9c-8622-34a81bf03b1a | Ubuntu_1804 | ACTIVE | sharednet1=10.0.0.202 | Ubuntu1804 | m1.small |
+--------------------------------------+-------------+--------+-----------------------+------------+----------+

ubuntu@dlp ~(keystone)$ ping 10.0.0.202 -c3
PING 10.0.0.202 (10.0.0.202) 56(84) bytes of data.
64 bytes from 10.0.0.202: icmp_seq=1 ttl=64 time=2.53 ms
64 bytes from 10.0.0.202: icmp_seq=2 ttl=64 time=0.892 ms
64 bytes from 10.0.0.202: icmp_seq=3 ttl=64 time=0.885 ms

--- 10.0.0.202 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 0.885/1.437/2.536/0.777 ms

ubuntu@dlp ~(keystone)$ ssh ubuntu@10.0.0.202
The authenticity of host '10.0.0.202 (10.0.0.202)' can't be established.
ECDSA key fingerprint is SHA256:EHQHwASJ6gE82uBv6TfFGpqVZeIHH8Wtj2gcFaZQJmY.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.0.0.202' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 18.04 LTS (GNU/Linux 4.15.0-23-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed Jun 13 16:24:04 JST 2018

  System load:  0.09              Processes:           89
  Usage of /:   19.0% of 8.80GB   Users logged in:     0
  Memory usage: 6%                IP address for ens3: 10.0.0.202
  Swap usage:   0%

0 packages can be updated.
0 updates are security updates.

Last login: Wed Jun 13 19:33:32 2018
ubuntu@ubuntu-1804:~$     # just logined```
[4]	If you'd like to stop an instance, it's also possible to control with openstack command like follows.
```
ubuntu@dlp ~(keystone)$ openstack server list
+--------------------------------------+-------------+--------+-----------------------+------------+----------+
| ID                                   | Name        | Status | Networks              | Image      | Flavor   |
+--------------------------------------+-------------+--------+-----------------------+------------+----------+
| deea0234-8e16-4f9c-8622-34a81bf03b1a | Ubuntu_1804 | ACTIVE | sharednet1=10.0.0.202 | Ubuntu1804 | m1.small |
+--------------------------------------+-------------+--------+-----------------------+------------+----------+

# stop instance
ubuntu@dlp ~(keystone)$ openstack server stop Ubuntu_1804
ubuntu@dlp ~(keystone)$ openstack server list
+--------------------------------------+-------------+---------+-----------------------+------------+----------+
| ID                                   | Name        | Status  | Networks              | Image      | Flavor   |
+--------------------------------------+-------------+---------+-----------------------+------------+----------+
| deea0234-8e16-4f9c-8622-34a81bf03b1a | Ubuntu_1804 | SHUTOFF | sharednet1=10.0.0.202 | Ubuntu1804 | m1.small |
+--------------------------------------+-------------+---------+-----------------------+------------+----------+

# start instance
ubuntu@dlp ~(keystone)$ openstack server start Ubuntu_1804
ubuntu@dlp ~(keystone)$ openstack server list
+--------------------------------------+-------------+--------+-----------------------+------------+----------+
| ID                                   | Name        | Status | Networks              | Image      | Flavor   |
+--------------------------------------+-------------+--------+-----------------------+------------+----------+
| deea0234-8e16-4f9c-8622-34a81bf03b1a | Ubuntu_1804 | ACTIVE | sharednet1=10.0.0.202 | Ubuntu1804 | m1.small |
+--------------------------------------+-------------+--------+-----------------------+------------+----------+```
[5]	It's possible to access with Web browser to get VNC console.
```
ubuntu@dlp ~(keystone)$ openstack server list
+--------------------------------------+-------------+--------+-----------------------+------------+----------+
| ID                                   | Name        | Status | Networks              | Image      | Flavor   |
+--------------------------------------+-------------+--------+-----------------------+------------+----------+
| deea0234-8e16-4f9c-8622-34a81bf03b1a | Ubuntu_1804 | ACTIVE | sharednet1=10.0.0.202 | Ubuntu1804 | m1.small |
+--------------------------------------+-------------+--------+-----------------------+------------+----------+

ubuntu@dlp ~(keystone)$ openstack console url show Ubuntu_1804
+-------+--------------------------------------------------------------------------------+
| Field | Value                                                                          |
+-------+--------------------------------------------------------------------------------+
| type  | novnc                                                                          |
| url   | http://10.0.0.30:6080/vnc_auto.html?token=48a51817-2bb4-4a4a-946f-4c2462709a59 |
+-------+--------------------------------------------------------------------------------+```
[6]	Access to the URL which was displayed by the command above.