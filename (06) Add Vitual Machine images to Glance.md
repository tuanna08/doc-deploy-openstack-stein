# (06) Add Vitual Machine images to Glance

	
Add Virtual Machine images to Glance.
[1]	
Install KVM HyperVisor on Compute Host, refer to here.
It's unnecessarry to set Bridge networking on the section [3] of the link.
[2]	For example, Create a Virtual Machine image of ubuntu 18.04.
```
# create a directory for disk image
root@dlp ~(keystone)# mkdir -p /var/kvm/images

# create a disk image
root@dlp ~(keystone)# qemu-img create -f qcow2 /var/kvm/images/ubuntu1804.img 10G
# install
root@dlp ~(keystone)# virt-install \
--name ubuntu1804 \
--ram 2048 \
--disk path=/var/kvm/images/ubuntu1804.img,format=qcow2 \
--vcpus 2 \
--os-type linux \
--os-variant ubuntu17.10 \
--network network=default \
--graphics none \
--console pty,target_type=serial \
--location 'http://ftp.riken.go.jp/Linux/ubuntu/dists/bionic/main/installer-amd64/' \
--extra-args 'console=ttyS0,115200n8 serial'
Starting install...     # installation starts
# after finishing installation, back to KVM Host and shutdown VM
root@dlp ~(keystone)# virsh shutdown ubuntu1804
# mount the disk image of the VM
root@dlp ~(keystone)# guestmount -d ubuntu1804 -i /mnt
# enable getty@ttyS0.service
root@dlp ~(keystone)# ln -s /mnt/lib/systemd/system/getty@.service /mnt/etc/systemd/system/getty.target.wants/getty@ttyS0.service
# unmount and start
root@dlp ~(keystone)# umount /mnt
root@dlp ~(keystone)# virsh start ubuntu1804 --console
# configure some settings for openstack instance on VM
root@ubuntu:~# apt-get update
root@ubuntu:~# apt-get -y install ssh cloud-init linux-virtual pollinate software-properties-common
root@ubuntu:~# vi /etc/default/grub
# line 11: change like follows
GRUB_DEFAULT=0
GRUB_HIDDEN_TIMEOUT=0
GRUB_HIDDEN_TIMEOUT_QUIET=true
GRUB_TIMEOUT=0
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT="console=tty1 console=ttyS0"
GRUB_CMDLINE_LINUX=""
# line 35,36: comment out
#GRUB_TERMINAL=serial
#GRUB_SERIAL_COMMAND="serial --unit=0 --speed=115200 --word=8 --parity=no --stopp=1"
root@ubuntu:~# update-grub
root@ubuntu:~# vi /etc/cloud/cloud.cfg
# line 13: add (if you'd like to allow SSH password authentication to login)
ssh_pwauth: true
# line 93: change (if you'd like to allow [ubuntu] user's SSH password auth)
default_user:
    name: ubuntu
    lock_passwd: False
root@ubuntu:~# systemctl enable serial-getty@ttyS0.service
# shutdown to finish settings
root@ubuntu:~# shutdown -h now
```
[3]	Add the Virtual Machine image to Glance.
```
root@dlp ~(keystone)# openstack image create "Ubuntu1804" --file /var/kvm/images/ubuntu1804.img --disk-format qcow2 --container-format bare --public
+------------------+------------------------------------------------------+
| Field            | Value                                                |
+------------------+------------------------------------------------------+
| checksum         | db7ba6799ed0ecbd1e75b32522e38500                     |
| container_format | bare                                                 |
| created_at       | 2018-06-13T04:40:23Z                                 |
| disk_format      | qcow2                                                |
| file             | /v2/images/bb285a87-fe42-449a-a659-b5a304177b18/file |
| id               | bb285a87-fe42-449a-a659-b5a304177b18                 |
| min_disk         | 0                                                    |
| min_ram          | 0                                                    |
| name             | Ubuntu1804                                           |
| owner            | 12d80218a35e4c62aa0403e92b5b649a                     |
| protected        | False                                                |
| schema           | /v2/schemas/image                                    |
| size             | 2980839424                                           |
| status           | active                                               |
| tags             |                                                      |
| updated_at       | 2018-06-13T04:40:42Z                                 |
| virtual_size     | None                                                 |
| visibility       | public                                               |
+------------------+------------------------------------------------------+

root@dlp ~(keystone)# openstack image list
+--------------------------------------+------------+--------+
| ID                                   | Name       | Status |
+--------------------------------------+------------+--------+
| bb285a87-fe42-449a-a659-b5a304177b18 | Ubuntu1804 | active |
+--------------------------------------+------------+--------+
[4]	By the way, if you got an image from internet to add it in Glance, it's OK to simply add it like follows.
root@dlp ~(keystone)# wget http://cloud-images.ubuntu.com/releases/18.04/release/ubuntu-18.04-server-cloudimg-amd64.img -P /var/kvm/images
root@dlp ~(keystone)# openstack image create "Ubuntu1804" --file /var/kvm/images/ubuntu-18.04-server-cloudimg-amd64.img --disk-format qcow2 --container-format bare --public
+------------------+------------------------------------------------------+
| Field            | Value                                                |
+------------------+------------------------------------------------------+
| checksum         | d7912161a35c8e8cc384589601751dd8                     |
| container_format | bare                                                 |
| created_at       | 2018-06-13T04:46:27Z                                 |
| disk_format      | qcow2                                                |
| file             | /v2/images/7e22891e-9d01-49b4-9eb2-4b02dbab051c/file |
| id               | 7e22891e-9d01-49b4-9eb2-4b02dbab051c                 |
| min_disk         | 0                                                    |
| min_ram          | 0                                                    |
| name             | Ubuntu1804                                           |
| owner            | 12d80218a35e4c62aa0403e92b5b649a                     |
| protected        | False                                                |
| schema           | /v2/schemas/image                                    |
| size             | 336461824                                            |
| status           | active                                               |
| tags             |                                                      |
| updated_at       | 2018-06-13T04:46:29Z                                 |
| virtual_size     | None                                                 |
| visibility       | public                                               |
+------------------+------------------------------------------------------+
```