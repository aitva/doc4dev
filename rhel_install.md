RHEL Server Configuration
===========================
This document is an addition to the centos_install.md document.
It highlight differences between RHEL and CentOS.
Do not hesitate to add or improve.

Register an RHEL machine
------------------------
If you want to use the package manager on a RHEL machine you must register.
To do so type: `# subscription-manager register --auto-attach`, you will be prompt
for your account name & password and one of the available license will be attached.
Below are useful commands using the subscription-manager:
```
# echo "attach machine to a specific license pool"
# subscription-manager attach --pool=POOL
#
# echo "list all available repositories"
# subscription-manager repos --list
#
# echo "enable a optional repository (php-fpm)"
# subscription-manager repos --enable rhel-server-rhscl-7-rpms
```

Configure yum to use a DVD on RHEL
--------------------------------
First, you need to mount the iso image:
```
# mkdir /mnt/rhel-server-7.2
# mount -rv -o loop rhel-server-7.2-x86_64-dvd.iso /mnt/rhel-server-7.2
```

Then, you need to configure yum to use this folder as a repository:
```
# cat > /etc/yum.repos.d/local.repo
[rhel-7-binary-dvd]
name = Red Hat Entreprise Linux 7.2 (DVD)
baseurl = file:///mnt/rhel-server-7.2
enabled = 1
gpgcheck = 1
gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release


[rhel-7-supp-dvd]
name = Red Hat Entreprise Linux 7.2 Supplementary (DVD)
baseurl = file:///mnt/rhel-supp-server-7.2
enabled = 1
gpgcheck = 1
gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
```

If you encounter slowness or strange errors while using yum try to disable
yum's plugins in `/etc/yum.conf` setting `plugins=0`.
