# Installation 
Thanks for your interest in Casa! Follow the instructions below to deploy an instance of Casa to offer end-users a self-service portal to manage their account security.  

View screenshots in the [User Guide](../user-guide.md).

## Gluu Server pre-requirements

Casa must be installed on the same server or VM as an operational Gluu Server 3.1.6 instance with at least the following components:  

- oxAuth OAuth2 Server        
- oxTrust GUI   
- Gluu LDAP  
- Apache     

**[Install Gluu 3.1.6](https://gluu.org/docs/ce/3.1.6/installation-guide/)**

### VM requirments
Make sure your VM or server has at least **1GB of additional RAM** on top of the [Gluu Server system requirements](https://gluu.org/docs/ce/3.1.6/installation-guide/#system-requirements) 

### Configuration requirements

- Dynamic client registration should be enabled in your Gluu Server before proceeding with Casa installation. After installation, dynamic client registration can be turned off as needed. [Read the docs](https://gluu.org/docs/ce/admin-guide/openid-connect/#dynamic-client-registration).

- If your Gluu Server 3.1.6 was upgraded from 2.4.4, ensure the `uma_protection` scope is allowed for dynamic registration in oxTrust. 

### User requirements
Casa shares the Gluu Server's data, so any user with an account in the Gluu Server will have an account in Casa. There are two types of users in Gluu Casa:

- **Admin users**: Admin users have access to the Casa [admin console](./admin-console.md). Any user in the `Managers Group` in the corresponding Gluu Server will be an Admin in Casa. 

- **Regular users**: Any other user in the Gluu Server is a "regular user" in Casa, and can use Casa to manage their account security preferences as outlined in the [user guide](../user-guide.md).  

Make sure you have an Admin user before proceeding with Casa installation. 

## Installation via Linux Packages 
Casa is distributed as part of the Gluu Server extensions bundle. Follow the instructions according to your underlying OS.

!!! Warning 
    If you have logged into the Gluu chroot in any console terminal, log out before proceeding.

### Ubuntu 14.04 (trusty)
    
| Command Description                   |               Trusty Commands         |
|---------------------------------------|---------------------------------------|
| Add Gluu Repository | `# echo "deb https://repo.gluu.org/ubuntu/ trusty main" > /etc/apt/sources.list.d/gluu-repo.list` |
| Add Gluu GPG Key                      | `# curl https://repo.gluu.org/ubuntu/gluu-apt.key | apt-key add -` |
| Update/Clean Repo                     | `# apt-get update`                    |
| Install Gluu Server extension pack    | `# apt-get install gluu-casa` |
 
### Ubuntu 16.04 (xenial)
      
|  Command Description    |               Xenial Commands         |
|-------------------------|---------------------------------------|
| Add Gluu Repository     | `# echo "deb https://repo.gluu.org/ubuntu/ xenial main" > /etc/apt/sources.list.d/gluu-repo.list` |
| Add Gluu GPG Key        | `# curl https://repo.gluu.org/ubuntu/gluu-apt.key | apt-key add -` |
| Update/Clean Repo       | `# apt-get update`                         |
| Install Gluu Server extension pack     | `# apt-get install gluu-casa`      |
 
### CentOS 6
      
| Command Description     |               CentOS 6.x              |
|-------------------------|---------------------------------------|
| Add Gluu Repository     | `# wget https://repo.gluu.org/centos/Gluu-centos6.repo -O /etc/yum.repos.d/Gluu.repo`|
| Add Gluu GPG Key        | `# wget https://repo.gluu.org/centos/RPM-GPG-KEY-GLUU -O /etc/pki/rpm-gpg/RPM-GPG-KEY-GLUU`|
| Import GPG Key          | `# rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-GLUU`|
| Update/Clean Repo       | `# yum clean all`                          |
| Install Gluu Server extension pack    | `# yum install gluu-casa`          |

### CentOS 7
     
| Command Description     |               CentOS 7.2              |
|-------------------------|---------------------------------------|
| Add Gluu Repository     | `# wget https://repo.gluu.org/centos/Gluu-centos7.repo -O /etc/yum.repos.d/Gluu.repo` |
| Add Gluu GPG Key        | `# wget https://repo.gluu.org/centos/RPM-GPG-KEY-GLUU -O /etc/pki/rpm-gpg/RPM-GPG-KEY-GLUU`|
| Import GPG Key          | `# rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-GLUU` |
| Update/Clean Repo       | `# yum clean all`                          |
| Install Gluu Server extension pack     | `# yum install gluu-casa`          |

### RHEL 6
     
| Command Description     |               RHEL 6.x              |
|-------------------------------|---------------------------------------|
| Add Gluu Repository     | `# wget https://repo.gluu.org/rhel/Gluu-rhel6.repo -O /etc/yum.repos.d/Gluu.repo` |
| Add Gluu GPG Key        | `# wget https://repo.gluu.org/centos/RPM-GPG-KEY-GLUU -O /etc/pki/rpm-gpg/RPM-GPG-KEY-GLUU`|
| Import GPG Key          | `# rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-GLUU` |
| Update/Clean Repo       | `# yum clean all`                          |
| Install Gluu Server extension pack     | `# yum install gluu-casa`          |

### RHEL 7
     
| Command Description     |               RHEL 7                  |
|-------------------------|---------------------------------------|
| Add Gluu Repository     | `# wget https://repo.gluu.org/rhel/Gluu-rhel7.repo -O /etc/yum.repos.d/Gluu.repo` |
| Add Gluu GPG Key        | `# wget https://repo.gluu.org/rhel/RPM-GPG-KEY-GLUU -O /etc/pki/rpm-gpg/RPM-GPG-KEY-GLUU`|
| Import GPG Key          | `# rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-GLUU` |
| Update/Clean Repo       | `# yum clean all`                          |
| Install Gluu Server extension pack     | `# yum install gluu-casa`          |

### Debian 8 (Jessie)

| Command Description     |               Jessie Commands         |
|-------------------------|---------------------------------------|
| Add Gluu Repository     | `# echo "deb https://repo.gluu.org/debian/ stable main" > /etc/apt/sources.list.d/gluu-repo.list`|
| Add Gluu GPG Key        | `# curl https://repo.gluu.org/debian/gluu-apt.key | apt-key add -` |
| Update/Clean Repo       | `# apt-get update`                         |
| Install Gluu Server extension pack     | `# apt-get install gluu-casa`      |

### Debian 9 (Stretch)

| Command Description     |               Jessie Commands         |
|-------------------------|---------------------------------------|
| Add Gluu Repository     | `# echo "deb https://repo.gluu.org/debian/ stable main" > /etc/apt/sources.list.d/gluu-repo.list`|
| Add Gluu GPG Key        | `# curl https://repo.gluu.org/debian/gluu-apt.key | apt-key add -` |
| Update/Clean Repo       | `# apt-get update`                         |
| Install Gluu Server extension pack     | `# apt-get install gluu-casa`      |

    
## Run the Setup Script

The Casa setup script, `setup_casa.py`, adds the application to the Gluu Server, imports required data to LDAP, and applies a number of required configurations in the Gluu Server chroot.

Log in to the Gluu Server chroot, as follows:

`$ service gluu-server-3.1.6 login`

(Or `gluu-serverd-3.1.6 start` for systemd-based distros). 

Then `cd` to the setup scripts directory and run `setup_casa.py`: 

```
# cd /install/community-edition-setup
# ./setup_casa.py
```

Answer the setup questions as prompted. Hit Enter to accept the default value specified in square brackets, if appropriate. 

## Finish setup
After answering the setup questions, your selections will be displayed with a prompt to finish installation. If everything looks good, hit Y to finish.

Upon successful installation, a confirmation message will appear that says: "Casa installation successful! Point your browser to `https://<host>/casa`".

Wait a couple of minutes, then visit the URL and authenticate against Gluu to access Casa. 

!!! Note
    As noted [above](#user-roles), make sure to sign in to Casa with a user that is included in the `Managers Group` in the Gluu Server. 

## Changing the URL path
To change the default URL path for Casa, follow the steps listed [here](change-context-path.md). It is advisable to apply this customization **before** credentials are enrolled. 
   
