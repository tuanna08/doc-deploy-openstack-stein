# (15) Configure Horizon
	
Configure OpenStack Dashboard Service (Horizon).
It's possible to control OpenStack on Web GUI to set Dashboard.
[1]	Install Horizon.
```
root@dlp ~(keystone)# apt -y install openstack-dashboard
```
[2]	Configure Horizon.
```
root@dlp ~(keystone)# vi /etc/openstack-dashboard/local_settings.py

# line 39 uncomment and add own hostname
ALLOWED_HOSTS = ['dlp.srv.world', 'localhost']
# line 65: uncomment like follows
OPENSTACK_API_VERSIONS = {
#    "data-processing": 1.1,
    "identity": 3,
    "image": 2,
    "volume": 2,
    "compute": 2,
}

# line 76: uncomment and change
OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
# line 98: uncomment
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = 'Default'
# line 163: change to your own Memcache server
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'LOCATION': '10.0.0.30:11211',
    },
}

# line 190: change to your own Host
OPENSTACK_HOST = "10.0.0.30"
OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "_member_"
root@dlp:~# systemctl restart apache2 memcached
```
[3]	
Access to the URL below with web browser.
⇒ http://(server's hostname or IP address)/horizon/
After accessing, following screen is displayed, then you can login with a user in Keystone.
It's possible to use all features if you login with admin user when you set it on keystone bootstrap. If you login with a common user, it's possible to use or manage own instances.