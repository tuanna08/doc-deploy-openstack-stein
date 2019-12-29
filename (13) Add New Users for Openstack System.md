# (13) Add New Users for Openstack System

	
Add Users in keystone who can use Openstack System.
[1]	Any names are OK you like for user-name or project-name.
Also Add flavors which define vCPU or Memory of an instance, too.
```
# add project
root@dlp ~(keystone)# openstack project create --domain default --description "Hiroshima Project" hiroshima
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Hiroshima Project                |
| domain_id   | default                          |
| enabled     | True                             |
| id          | 78d261e9ec6c484b927a6006e780306c |
| is_domain   | False                            |
| name        | hiroshima                        |
| parent_id   | default                          |
| tags        | []                               |
+-------------+----------------------------------+

# add user
root@dlp ~(keystone)# openstack user create --domain default --project hiroshima --password userpassword serverworld
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| default_project_id  | 78d261e9ec6c484b927a6006e780306c |
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 7992021fa3e34c9bbe2d86f5beebd01c |
| name                | serverworld                      |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

# add role
root@dlp ~(keystone)# openstack role create CloudUser
+-----------+----------------------------------+
| Field     | Value                            |
+-----------+----------------------------------+
| domain_id | None                             |
| id        | dd776335b84748d9954c8e1f5a660cc2 |
| name      | CloudUser                        |
+-----------+----------------------------------+

# add user to the role
root@dlp ~(keystone)# openstack role add --project hiroshima --user serverworld CloudUser
# add flavor
root@dlp ~(keystone)# openstack flavor create --id 0 --vcpus 1 --ram 2048 --disk 10 m1.small
+----------------------------+----------+
| Field                      | Value    |
+----------------------------+----------+
| OS-FLV-DISABLED:disabled   | False    |
| OS-FLV-EXT-DATA:ephemeral  | 0        |
| disk                       | 10       |
| id                         | 0        |
| name                       | m1.small |
| os-flavor-access:is_public | True     |
| properties                 |          |
| ram                        | 2048     |
| rxtx_factor                | 1.0      |
| swap                       |          |
| vcpus                      | 1        |
+----------------------------+----------+```