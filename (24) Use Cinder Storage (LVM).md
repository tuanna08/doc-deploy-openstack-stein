# (24) Use Cinder Storage (LVM)

It's possible to use Virtual Storages provided by Cinder if an Instance needs more disks.
Configure Virtual storage with LVM backend on here.
It needs there are some free spaces on disks of Storage Node.
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
     |  Nova API             |   |     Cinder-Volume     |   |                       |
     |  Neutron Server       |   |                       |   |                       |
     |  Metadata Agent       |   |                       |   |                       |
     |  Cinder API           |   |                       |   |                       |
     +-----------------------+   +-----------------------+   +-----------------------+
```

[1]	Create a volume group for Cinder on Storage Node.
```
root@storage:~# pvcreate /dev/sdb1
Physical volume "/dev/sdb1" successfully created
root@storage:~# vgcreate -s 32M vg_volume01 /dev/sdb1
Volume group "vg_volume01" successfully created
```
[2]	Configure Cinder Volume on Storage Node.
```
root@storage:~# apt -y install tgt thin-provisioning-tools
root@storage:~# vi /etc/cinder/cinder.conf
# add a value for enabled_backends
enabled_backends = lvm
# add follows to the end
[lvm]
iscsi_helper = tgtadm
# volume group name just created
volume_group = vg_volume01
# IP address of Storage Node
iscsi_ip_address = 10.0.0.50
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volumes_dir = $state_path/volumes
iscsi_protocol = iscsi
root@storage:~# systemctl restart cinder-volume tgt
```
[3]	Configure Nova on Compute Node.
```
root@node01:~# vi /etc/nova/nova.conf
# add to the end
[cinder]
os_region_name = RegionOne
root@node01:~# systemctl restart nova-compute
```
[4]	Login as a common user you'd like to add volumes to own instances.
```
For example, create a virtual disk [disk01] with 10GB. It's OK to work on any node. (This example is on Control Node)
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
| created_at          | 2018-06-18T03:55:19.000000           |
| description         | None                                 |
| encrypted           | False                                |
| id                  | b253402e-806d-4ded-a1f7-027636a2b567 |
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
| b253402e-806d-4ded-a1f7-027636a2b567 | disk01 | available |   10 |             |
+--------------------------------------+--------+-----------+------+-------------+
```
[5]	Attach the virtual disk to an Instance.
```
For the exmaple below, the disk is connected as [/dev/vdb]. It's possible to use it as a storage to create a file system on it.
ubuntu@dlp ~(keystone)$ openstack server list
+-------------+-------------+---------+-----------------------------------+------------+----------+
| ID          | Name        | Status  | Networks                          | Image      | Flavor   |
+-------------+-------------+---------+-----------------------------------+------------+----------+
| 3aae02a-... | Ubuntu_1804 | SHUTOFF | int_net=192.168.100.5, 10.0.0.205 | Ubuntu1804 | m1.small |
+-------------+-------------+---------+-----------------------------------+------------+----------+

ubuntu@dlp ~(keystone)$ openstack server add volume Ubuntu_1804 disk01
# the status of attached disk turns [in-use] like follows
ubuntu@dlp ~(keystone)$ openstack volume list
+--------------------------------------+--------+--------+------+--------------------------------------+
| ID                                   | Name   | Status | Size | Attached to                          |
+--------------------------------------+--------+--------+------+--------------------------------------+
| b253402e-806d-4ded-a1f7-027636a2b567 | disk01 | in-use |   10 | Attached to Ubuntu_1804 on /dev/vdb  |
+--------------------------------------+--------+--------+------+--------------------------------------+

# for detaching the disk, run like follows
ubuntu@dlp ~(keystone)$ openstack server remove volume Ubuntu_1804 disk01
```