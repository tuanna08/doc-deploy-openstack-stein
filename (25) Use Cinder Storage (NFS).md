# (25) Use Cinder Storage (NFS)


It's possible to use Virtual Storages provided by Cinder if an Instance needs more disks.
Configure Virtual storage with NFS backend on here.
```
------------+---------------------------+---------------------------+-------------------------+------------
            |                           |                           |                         |
        eth0|10.0.0.30              eth0|10.0.0.50              eth0|10.0.0.51            eth0|10.0.0.35
+-----------+-----------+   +-----------+-----------+   +-----------+-----------+  +----------+-----------+
|    [ Control Node ]   |   |    [ Storage Node ]   |   |    [ Compute Node ]   |  |   [  NFS Server  ]   |
|                       |   |                       |   |                       |  |                      |
|  MariaDB    RabbitMQ  |   |                       |   |        Libvirt        |  +----------------------+
|  Memcached  httpd     |   |        L2 Agent       |   |     Nova Compute      |
|  Keystone   Glance    |   |        L3 Agent       |   |        L2 Agent       |
|  Nova API             |   |     Metadata Agent    |   |                       |
|  Neutron Server       |   |     Cinder-Volume     |   |                       |
|  Metadata Agent       |   |                       |   |                       |
|  Cinder API           |   |                       |   |                       |
+-----------------------+   +-----------------------+   +-----------------------+
```

[1]	NFS server is required to be running on your LAN, refer to here.
On this example, configure [/var/lib/nfs-share] directory on [nfs.srv.world] as a shared directory.
[2]	Configure Storage Node.
```
root@storage:~# apt -y install nfs-common
root@storage:~# vi /etc/idmapd.conf
# line 6: uncomment and change to the own domain name
Domain = srv.world
root@storage:~# vi /etc/cinder/cinder.conf
# add the value for enabled_backends
enabled_backends = nfs
# add follows to the end
[nfs]
volume_driver = cinder.volume.drivers.nfs.NfsDriver
nfs_shares_config = /etc/cinder/nfs_shares
nfs_mount_point_base = $state_path/mnt
root@storage:~# vi /etc/cinder/nfs_shares
# create new : specify NFS shared directory
nfs.srv.world:/var/lib/nfs-share
root@storage:~# chmod 640 /etc/cinder/nfs_shares
root@storage:~# chgrp cinder /etc/cinder/nfs_shares
root@storage:~# systemctl restart cinder-volume
root@storage:~# chown -R cinder. /var/lib/cinder/mnt
```
[3]	Change Nova settings on Compute Node to mount NFS.
```
root@node01:~# apt -y install nfs-common
root@node01:~# vi /etc/idmapd.conf
# line 6: uncomment and change to the own domain name
Domain = srv.world
root@node01:~# vi /etc/nova/nova.conf
# add to the end
[cinder]
os_region_name = RegionOne
root@node01:~# systemctl restart nova-compute
```
[4]	Login as a common user you'd like to add volumes to own instances.
For example, create a virtual disk [disk01] with 10GB. It's OK to work on any node. (This example is on Control Node)
```
# set environment variable first
ubuntu@dlp ~(keystone)$ echo "export OS_VOLUME_API_VERSION=2" >> ~/keystonerc
ubuntu@dlp ~(keystone)$ source ~/keystonerc
ubuntu@dlp ~(keystone)$ openstack volume create --size 10 disk01
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| attachments         | []                                   |
| availability_zone   | nova                                 |
| bootable            | false                                |
| consistencygroup_id | None                                 |
| created_at          | 2018-06-18T04:45:12.000000           |
| description         | None                                 |
| encrypted           | False                                |
| id                  | facf1411-a20f-4885-8535-e5758ab1a6d2 |
| multiattach         | False                                |
| name                | disk01                               |
| properties          |                                      |
| replication_status  | None                                 |
| size                | 10                                   |
| snapshot_id         | None                                 |
| source_volid        | None                                 |
| status              | creating                             |
| type                | None                                 |
| updated_at          | None                                 |
| user_id             | 601473fae2c444a484860046ef2484e2     |
+---------------------+--------------------------------------+

ubuntu@dlp ~(keystone)$ openstack volume list
+--------------------------------------+--------+-----------+------+-------------+
| ID                                   | Name   | Status    | Size | Attached to |
+--------------------------------------+--------+-----------+------+-------------+
| facf1411-a20f-4885-8535-e5758ab1a6d2 | disk01 | available |   10 |             |
+--------------------------------------+--------+-----------+------+-------------+
```
[5]	Attach the virtual disk to an Instance.
For the exmaple below, the disk is connected as [/dev/vdb]. It's possible to use it as a storage to create a file system on it.
```
ubuntu@dlp ~(keystone)$ openstack server list
+------------+-------------+---------+-----------------------------------+------------+----------+
| ID         | Name        | Status  | Networks                          | Image      | Flavor   |
+------------+-------------+---------+-----------------------------------+------------+----------+
| 3aae02a... | Ubuntu_1804 | SHUTOFF | int_net=192.168.100.5, 10.0.0.205 | Ubuntu1804 | m1.small |
+------------+-------------+---------+-----------------------------------+------------+----------+

ubuntu@dlp ~(keystone)$ openstack server add volume Ubuntu_1804 disk01
# the status of attached disk turns [in-use] like follows
ubuntu@dlp ~(keystone)$ openstack volume list
+--------------------------------------+--------+--------+------+--------------------------------------+
| ID                                   | Name   | Status | Size | Attached to                          |
+--------------------------------------+--------+--------+------+--------------------------------------+
| facf1411-a20f-4885-8535-e5758ab1a6d2 | disk01 | in-use |   10 | Attached to Ubuntu_1804 on /dev/vdb  |
+--------------------------------------+--------+--------+------+--------------------------------------+

# detach the disk
ubuntu@dlp ~(keystone)$ openstack server remove volume Ubuntu_1804 disk01
```