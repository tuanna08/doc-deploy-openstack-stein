# (01) Install KVM


This is the Virtualization Configuration with KVM ( Kernel-based Virtual Machine ) + QEMU.
This requires that the CPU on your computer which has a function Intel VT or AMD-V.
[1]	Install required packages.

```
root@dlp:~# apt -y install qemu-kvm libvirt-bin virtinst bridge-utils libosinfo-bin libguestfs-tools virt-top
```
[2]	Enable vhost-net.

```
root@dlp:~# modprobe vhost_net
root@dlp:~# lsmod | grep vhost
vhost_net              20480  0
vhost                  32768  1 vhost_net
macvtap                20480  1 vhost_net
root@dlp:~# echo vhost_net >> /etc/modules
```
[3]	Configure Bridge networking.
```
root@dlp:~# vi /etc/netplan/01-netcfg.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens3:
      dhcp4: no
      # disable existing configuration for ethernet
      #addresses: [10.0.0.30/24]
      #gateway4: 10.0.0.1
      #nameservers:
        #addresses: [10.0.0.10]
      dhcp6: no
  # add configuration for bridge interface
  bridges:
    br0:
      interfaces: [ens3]
      dhcp4: no
      addresses: [10.0.0.30/24]
      gateway4: 10.0.0.1
      nameservers:
        addresses: [10.0.0.10]
      parameters:
        stp: false
      dhcp6: no

root@dlp:~# reboot
root@dlp:~# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master br0 state UP group default qlen 1000
    link/ether 52:54:00:ac:76:41 brd ff:ff:ff:ff:ff:ff
3: br0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether aa:b3:1c:89:71:0e brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.30/24 brd 10.0.0.255 scope global br0
       valid_lft forever preferred_lft forever
    inet6 fe80::a8b3:1cff:fe89:710e/64 scope link
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 52:54:00:cb:78:ae brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr0 state DOWN group default qlen 1000
    link/ether 52:54:00:cb:78:ae brd ff:ff:ff:ff:ff:ff
    ```