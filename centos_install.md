CentOS Server Configuration
===========================
Installing a CentOS server from scratch takes time.
This document is a collection of hints and commands for future use.
Do not hesitate to add or improve.

System partitions
-----------------
Here is the partitions' configuration for the CAMS' Norway servers :

mount point | size (GiB)
------------|-----------
/           | 60
/home       | 20
/opt        | 40
/var        | 150
SWAP        | 12.5
BOOT        | 0.5

Additional disks
----------------
Below a list of commands to format and mount additional disks:
- `# fdisk -l`
- `# fdisk /dev/sdb` (type `m` for help)
- `# mkfs.xfs`
- `# mount /dev/sdb1 /images`
- `# vi /etc/fstab` (the zeros in the lasts column are historic)

Network configuration
---------------------
Network scripts are located in `/etc/sysconfig/network-scripts/` and are name
`ifcfg-XXX` where XXX replace the interface name.

Below is an example configuration file for `ifcfg-XXX` using DHCP:
```
DEVICE=XXX # Must be equivalent to ifcfg-XXX
BOOTPROTO=dhcp
ONBOOT=yes
```

Below is an example configuration file for `ifcfg-XXX`:
```
DEVICE=XXX # Must be equivalent to ifcfg-XXX
BOOTPROTO=no
ONBOOT=yes
IPADDR=192.168.32.6
NETMASK=255.255.255.0
GATEWAY=192.168.32.2
```

Configure yum to use DVD on CentOS
----------------------------------
Yum repositories configuration's files are located into `/etc/yum.repos.d`. To activate
the DVD repository edit `/etc/yum.repos.d/CentOS-Media.repo` and set `enable=1`.
Below is a command to force the media repository :
```
# yum --disablerepo=\* --enablerepo=c7-media install pkg-name
```

Essential packages
------------------
Below is a list of the minimal package to have a functional HTTP server:
- `policycoreutils-python`
- `httpd`
- `mariadb mariadb-server`
- `php-fpm php-mysqlnd php-pdo`
- `vim nc tree`

Firewall configuration
----------------------
Below are the command to allow HTTP traffic:
```
# firewall-cmd --permanent --zone=public --add-service=http
# firewall-cmd --permanent --zone=public --add-service=https
```

Enable server on BOOT
---------------------
CentOS does not automatically add applications to the default boot process.
You will need to do it explicitly:
```
# systemctl enable httpd
# systemctl enable mariadb
# systemctl enable php-fpm
```

MySQL Configuration
-------------------
Below a command to setup a minimal & secure MySQL config:
```
# /usr/bin/mysql-secure-installation
```

HTTPd Configuration
-------------------
To setup a minimal Apache configuration you may want to:
- comment the content of `/etc/httpd/conf.d/welcome.conf`
- change the default mpm_event_module in `/etc/httpd/conf.modules.d/00-mpm.conf`

Below is the configuration file installed on the CAMS' Norway server:
```
# Solystic's configuration file.
#
# Author: L. Arod
# Date: 2016/03/03
#
# To improve performances please open /etc/httpd/conf.modules.d/00-mpm.conf
# and uncomment:
#     LoadModule mpm_worker_module modules/mod_mpm_worker.so

DocumentRoot "/opt/solystic/services"
<Directory "/opt/solystic/services">
    AllowOverride None
    Require all granted
    Options +Indexes
</Directory>

# Improve log formating.
# Author: H. Lagrange
LogFormat "%h %{begin:%H:%M:%S}t.%{begin:msec_frac}t duration=%D (us) rec=%I (bytes) \"%r\" %>s sent=%O (bytes)" common
LogFormat "%h %{begin:%H:%M:%S}t.%{begin:msec_frac}t pid=(%{pid}P %{tid}P) duration=%D (us) rec=%I (bytes) \"%r\" %>s sent=%O (bytes)" combined

# Enable apache status page.
ExtendedStatus On
<Location /server-status>
    SetHandler server-status
    Order Allow,Deny
    Allow from all
</Location>

# Improve MPM Worker perf.
<IfModule mpm_worker_module>
    StartServers          8
    MinSpareThreads      32
    MaxSpareThreads      64
    ThreadLimit          32
    ThreadsPerChild      32
    MaxClients          512
    MaxRequestsPerChild   0
</IfModule>

# Pass request to PHP-FPM.
ProxyPassMatch "^/(php-status|ping)" "unix:/var/run/php5-fpm.sock|fcgi://localhost/"
ProxyPassMatch "^/.*\.php(/.*)?$" "unix:/var/run/php5-fpm.sock|fcgi://localhost/opt/solystic/services" enablereuse=On flushpackets=On

```

PHP-FPM configuration
---------------------
Below is a list of fields change in `/etc/php-fpm.d/www.pool` :
```
listen = /var/run/php5-fpm.sock

pm.max_children = 200
pm.start_servers = 60
pm.min_spare_servers = 30
pm.max_spare_servers = 60
pm.status_path = /php-status

ping.path = /ping
ping.response = pong
```

Installing SVN
--------------
To install svn on development machine you need to run the following commands:
```
# yum install subversion cyrus-sasl-md5
```

SELinux Bonus
-------------
Here are some useful command to work with SELinux:
- `# chcon --type var_t /var/www/html/index.html`
- `# semanage fcontext --add --type httpd_sys_content_t "/www(/.*)?"`
- `# restorecon -Rv /www`
