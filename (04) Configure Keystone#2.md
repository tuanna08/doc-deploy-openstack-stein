# (04) Configure Keystone#2
	
Add Projects in Keystone.
This example is based on the environment like follows.
        eth0|10.0.0.30 
+-----------+-----------+
|    [ Control Node ]   |
|                       |
|  MariaDB    RabbitMQ  |
|  Memcached  httpd     |
|  Keystone             |
+-----------------------+

[1]	Load environment variables first.
The value for [OS_PASSWORD] is from the password when configuring keystone bootstrap.
For [OS_AUTH_URL], specify Keystone server's hostname or IP address.

```
root@dlp:~# vi ~/keystonerc
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=adminpassword
export OS_AUTH_URL=http://10.0.0.30:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
export PS1='\u@\h \W(keystone)\$ '
root@dlp:~# chmod 600 ~/keystonerc
root@dlp:~# source ~/keystonerc
root@dlp ~(keystone)# echo "source ~/keystonerc " >> ~/.bash_profile
```

[2]	Add Projects.
```

# add service project
root@dlp ~(keystone)# openstack project create --domain default --description "Service Project" service
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Service Project                  |
| domain_id   | default                          |
| enabled     | True                             |
| id          | efc5c6a05d444cae80c0a80f49ba87d4 |
| is_domain   | False                            |
| name        | service                          |
| parent_id   | default                          |
| tags        | []                               |
+-------------+----------------------------------+

# confirm settings
root@dlp ~(keystone)# openstack project list
+----------------------------------+---------+
| ID                               | Name    |
+----------------------------------+---------+
| e52584cf60234bc090c7d437b8ca7b62 | admin   |
| efc5c6a05d444cae80c0a80f49ba87d4 | service |
+----------------------------------+---------+
```