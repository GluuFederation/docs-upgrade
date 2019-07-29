# Installation 
## Overview
There are Gluu Server Linux packages for Ubuntu, CentOS, RHEL and Debian operating systems. The installation procedure is similar across all distributions: 

1. [Install the Linux package](#install-the-package)
2. [Start the Server and log in to the container](#start-the-server-and-log-in)
3. [Run `setup.py`](#run-setuppy)
4. [Sign in via browser](#sign-in-via-browser)
5. [Disable Gluu repositories](#disable-gluu-repositories)

!!! Note
    The below instructions are intended for single server Gluu deployments. If you intend to cluster your Gluu Server to achieve fail-over and high availability, please refer to the [cluster documentation](./cluster.md)

## Prerequisites
Make sure the target server or VM meets **all minimum requirements** as specified in the [VM Preparation Guide](../installation-guide/index.md).   

There are a few system specific notes to follow:  

- **CentOS**: selinux must be set to permissive in /etc/selinux/config

- **Debian**: make sure the `apt-transport-https` package is already installed on the target system before the `gluu-repo.list` is added. Otherwise the installation might be hindered.

- **Linux containers (e.g. Docker)**: This guide does not support installation via Linux containers. See [Gluu Server Docker Edition (DE)](https://gluu.org/docs/de) documentation for detailed instructions.
  

## Instructions

### Install the package

Installation of the Gluu server will be done under `/root`. 
The Gluu Server will create its file system under `/root/` and will be installed under `/opt`. File size and [minimum requirements](../installation-guide/index.md) remain the same as the host.


#### Ubuntu Server 18.04.x

```
echo "deb https://repo.gluu.org/ubuntu/ bionic-devel main" > /etc/apt/sources.list.d/gluu-repo.list
```

```
curl https://repo.gluu.org/ubuntu/gluu-apt.key | apt-key add -
```

```
apt-get update
```

```
apt-get install gluu-server-4.0
```

!!! Note  
    If you use the server version of Ubuntu 18, you need to add the Ubuntu Universe repositories as well. Use these commands: `# echo "deb http://archive.ubuntu.com/ubuntu bionic universe" >> /etc/apt/sources.list` and `# echo "deb http://archive.ubuntu.com/ubuntu bionic-updates universe" >> /etc/apt/sources.list` 
  

#### Ubuntu Server 16.04.x

```
echo "deb https://repo.gluu.org/ubuntu/ xenial-devel main" > /etc/apt/sources.list.d/gluu-repo.list
```

```
curl https://repo.gluu.org/ubuntu/gluu-apt.key | apt-key add -
```

```
apt-get update
```

```
apt-get install gluu-server-4.0
```

#### CentOS 7.x

```
wget https://repo.gluu.org/centos/Gluu-centos-7-testing.repo -O /etc/yum.repos.d/Gluu.repo
```

```
wget https://repo.gluu.org/centos/RPM-GPG-KEY-GLUU -O /etc/pki/rpm-gpg/RPM-GPG-KEY-GLUU
```

```
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-GLUU
```

```
yum clean all
```

```
yum install gluu-server-4.0
```

#### RHEL 7.x

```
wget https://repo.gluu.org/rhel/Gluu-rhel-7-testing.repo -O /etc/yum.repos.d/Gluu.repo
```

```
wget https://repo.gluu.org/rhel/RPM-GPG-KEY-GLUU -O /etc/pki/rpm-gpg/RPM-GPG-KEY-GLUU
```

```
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-GLUU
```

```
yum clean all
```

```
yum install gluu-server-4.0
```

#### Debian 9 (Stretch)

```
echo "deb https://repo.gluu.org/debian/ stretch-testing main" > /etc/apt/sources.list.d/gluu-repo.list
```

```
# curl https://repo.gluu.org/debian/gluu-apt.key | apt-key add -
```

```
apt-get update
```

```
apt-get install gluu-server-4.0
```

### Start the server and log in

The Gluu Server is a chroot container, which must be started to proceed. 

For Centos 7.x, Red Hat 7.x, Ubuntu 18 and Debian 9, run the following commands: 

```
/sbin/gluu-serverd-4.0 enable
```

```
/sbin/gluu-serverd-4.0 start
```

```
/sbin/gluu-serverd-4.0 login
```

!!! Note
    Only use `enable` the first time you start the Gluu Server.

For Ubuntu 16, run the following commands:

```
service gluu-server-4.0 start
```

```
service gluu-server-4.0 login
```

### Run `setup.py`

Configuration is completed by running `setup.py` from inside the chroot container. This generates certificates, salt values, and renders configuration files. 

```
cd /install/community-edition-setup
```

```
./setup.py
```   

A prompt will appear to answer a few questions about the deployment. Hit `Enter` to accept the default values. 

Refer to the following table for details about available setup options:    

| Setup Option                |  Explanation                               |
|-------------------------|--------------------------------------------|
| Enter IP Address | Used primarily by Apache httpd for the [Listen](https://httpd.apache.org/docs/2.4/bind.html) directive. **Use an IP address assigned to one of this server's network interfaces (usage of addresses assigned to loopback interfaces is not supported)**|
| Enter hostname | Internet-facing FQDN that is used to generate certificates and metadata. **Do not use an IP address or localhost.** |
| Enter your city or locality | Used to generate X.509 certificates. |
| Enter your state or province two letter code | Used to generate X.509 certificates. |
| Enter two letter Country Code | Used to generate X.509 certificates. |
| Enter Organization Name | Used to generate X.509 certificates. |
| Enter email address for support at your organization | Used to generate X.509 certificates. | 
| Optional: enter password for oxTrust and LDAP superuser | Used as the LDAP directory manager password, and for the default admin user for oxTrust. |
| Install oxAuth OAuth2 Authorization Server | Required. Includes Gluu's OpenID Connect provider (OP) and UMA authorization server (AS) implementations.|
| Install oxTrust Admin UI | Required. This is the Gluu server admin dashboard. |
| Install Backend DB Server | Required. Installs OpenDJ, used to store user info and configuration data. |
| Install Apache 2 web server | Required |
| Install Shibboleth SAML IDP | Optional. Only install if a SAML identity provider (IDP) is needed. |
| Install oxAuth RP | Optional. OpenID Connect test client: useful for test environments, for more details see [here](../admin-guide/openid-connect/#oxauth-rp) |
| Install Passport |  Optional. Install if you want to support external IDP, for instance to offer users social login. |
| Install Gluu Radius | Optional. Installs Radius server. More information is available [here](../admin-guide/radius-server/gluu-radius.md)

When complete, `setup.py` will show the selections and prompt for confirmation. If everything looks OK, select Y to finish installation. 

After 5-10 minutes the following success message will appear: 

`Gluu Server installation successful! Point your browser to [hostname].`


#### Avoiding common issues

Avoid setup issues by acknowledging the following:         

- IP Address: Do **not** use `localhost` for either the IP address or hostname.     

- Hostname:     
     - Make sure to choose the hostname carefully. Changing the hostname after installation is not a simple task.   
     - Use a real hostname--this can always be managed via host file entries if adding a DNS entry is too much work for testing.   
     - For clustered deployments, use the hostname of the cluster that will be used by applications connecting to Gluu.   

- Only run setup.py **one time**. Running the command twice will break the instance.

### Sign in via browser

Wait about 10 minutes in total for the server to restart and finalize its configuration. After that period, sign in via a web browser. The username will be `admin` and your password will be the `ldap_password` you provided during installation. 

!!! Note   
    If the Gluu Server login page does not appear, confirm that port 443 is open in the VM. If it is not open, open port 443 and try to reach the host in the browser again.   

### Disable Gluu Repositories

To prevent involuntary overwrites of the currently deployed instance (in case a newer version of the same package is found during regular OS updates), disable the previously added Gluu repositories after initial installation.

For CentOS/RHEL: 

`/etc/yum.repos.d/Gluu.repo` needs to be edited so that the `enabled=1` clause is changed to `enabled=0`        

For Ubuntu/Debian: 

`/etc/apt/sources.list.d/gluu-repo.list` needs to be edited to comment out all Gluu-related repos.     

!!! Note
    The Gluu Server does **not** support package updates/upgrades via Linux package management (i.e. using commands like `# yum update` or `# apt-get update`). For upgrade instructions, see the [upgrade docs](../upgrade/index.md).


## Backups
Sometimes things go wrong! It can be difficult to troubleshoot issues if the steps to reproduce the issue are not clearly documented. This is why we **always** recommend creating [backups of your Gluu Server](../operation/backup.md). 

## Uninstallation

###  Uninstall Ubuntu Server 18.04.x

```
/sbin/gluu-serverd-4.0 disable
```

```
/sbin/gluu-serverd-4.0 stop
```

```
apt-get remove gluu-server-4.0
```

```
rm -fr /opt/gluu-server.save
```

### Uninstall Ubuntu Server 16.4.x or Debian 9 (Stretch):

```
service gluu-server-4.0 stop
```

```
apt-get remove gluu-server-4.0
```

```
rm -rf /opt/gluu-server-4.0.save
```

### Uninstall Centos 7.x and Red Hat 7.x

```
/sbin/gluu-serverd-4.0 disable
```

```
/sbin/gluu-serverd-4.0 stop
```

```
yum remove gluu-server-4.0 
```

```
rm -rf /opt/gluu-server-4.0.save
```

!!! Note
    `apt-get purge gluu-server-4.0` or `apt-get remove --purge gluu-server-4.0` can also be used to uninstall and remove all the folders and services of the Gluu Server. Make sure to back up ALL directories of `/opt` into other direction (tmp or root directory itself) before running the purge command.

## Support
Please review the [Gluu support portal](https://support.gluu.org). There are many existing tickets about troubleshooting installation issues. If there is no similar existing public issue, register for an account and open a new ticket. 

If your organization needs guaranteed responses, SLAs, and priority access to the Gluu support and development team, consider purchasing one of our [VIP support contracts](https://gluu.org/pricing).  
