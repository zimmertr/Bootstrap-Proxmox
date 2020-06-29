# TKS-Bootstrap_Proxmox

* [Summary](#Summary)
* [How To Use](#How-To-Use)
* [Instructions](#Instructions)
   * [Install Proxmox VE](#Install-Proxmox-VE)
   * [Create User Account](#Create-User-Account)
   * [Configure Proxmox](#Configure-Proxmox)
   * [Configure Clustering](#Configure-Clustering)
   * [Configure Storage](#Configure-Storage)

## Summary

`Bootstrap_Proxmox` sets up a Proxmox server for TKS by creating the necessary user accounts, installing package dependencies, and more. 

## How To Use

This repository can be used on its own but it is intended to be used as a submodule of [TKS](https://github.com/zimmertr/TKS). Consider cloning that instead. TKS enable enthusiasts and administrators alike to easily provision highly available and production-ready Kubernetes clusters on Proxmox VE.

Ansible is used to configure Proxmox. Logic is split into multiple roles which are often highly configurable and even optional. Configurations are applied to TKS using environment variables. For a list of supported environment variables, see the README document for each role. 

## Instructions

### Install Proxmox VE

In my case, I have both a Dell server with IDRAC and a Mac Pro that requires a bootable USB drive. So instructions for both methods will be provided. After booting from the medium, proceed to install Proxmox as usual. 

**Dell IDRAC:**

1. *Download the latest ISO for [Proxmox VE](https://www.proxmox.com/en/downloads/category/iso-images-pve).*
2. *Connect to IDRAC and launch a new Virtual Console.*
3. *Click `Virtual Media` and select your downloaded Proxmox VE ISO as a new CD/DVD Image File.* 
4. *Click `Map Device` and reboot the server. Interrupt the boot process by tapping `F10` to access the Dell Lifecycle Controller.*
5. *Navigate through `OS Deployment` -> `Deploy OS` -> `Go Directly to OS Deployment`* 
6. *Set `Available Operation Systems` to `Any Other Operation System`, choose a `Manual Install`, choose `PVE Virtual CD` as your media,* 
7. *Press `Finish` and wait for the Lifecycle Controller to boot into the Proxmox VE installer.* 

**Bootable USB:**

1. *Connect a flash drive to your workstation and use [fdisk](https://linux.die.net/man/8/fdisk) or [diskutil](https://ss64.com/osx/diskutil.html) to determine the mountpath. Mine is `/dev/sdf`.*

2. *Export the required variables documented in the role's README.* 

   ```bash
   export TKS_BM_V_PROXMOX_ISO_URL="https://www.proxmox.com/en/downloads?task=callelement&format=raw&item_id=513&element=f85c494b-2b32-4109-b8c1-083cca2b7db6&method=download&args[0]=e20c5339a85f415aa8786ae730d14f05"
   export TKS_BM_V_USB_MOUNTPATH=/dev/sdf
   ```

3. *Execute the role using the `create_usb_medium.yml` playbook  and eject the flash drive when finished.* 

   ```bash
   ansible-playbook create_usb_medium.yml
   sudo eject $TKS_BM_V_USB_MOUNTPATH
   ```

4. *Disconnect the flash drive from your workstation and connect it to your server. Power it on, and boot from the USB. With my Mac Pro, I accomplish this by holding down the `Option` key. Allow the computer to boot into the Proxmox VE installer.* 

<hr>


### Create a new user account

Now that Proxmox has been installed, it's time to set up a user account for Ansible to use. We'll also be creating an SSH key and adding it to the `authorized_keys` file for that user as well as installing `sudo` and making them a sudoer. 

1. *Export the `ANSIBLE_REMOTE_USER` and `ANSIBLE_ASK_PASS` environment variables. This is necessary at first since Proxmox does not yet have an SSH key.*

   ```bash
   export ANSIBLE_REMOTE_USER=root
   export ANSIBLE_ASK_PASS=true
   ```

2. *In order to use password authentication with Ansible, you will need to also install the [sshpass](https://linux.die.net/man/1/sshpass) package.*

   ```bash
   sudo pacman -S sshpass
   ```

3. *Create a new SSH Key and User Account for Ansible to use*

   ```bash
   export TKS_BP_R_CREATE_USER_ACCOUNT=true
   export TKS_BP_V_PROXMOX_SSH_KEY='~/.ssh/sol.milkyway'
   export TKS_BP_V_PROXMOX_USER_NAME=tj    
   
   ansible-playbook -i inventory.yml TKS-Bootstrap_Proxmox/Ansible/create_user_account.yml
   ```

4. *Reconfigure Ansible to use SSH keys for authentication as well as your new user account.*

   ```bash
   export ANSIBLE_REMOTE_USER=tj
   export ANSIBLE_ASK_PASS=false
   export ANSIBLE_PRIVATE_KEY_FILE=~/.ssh/sol.milkyway
   ```

<hr>


### Configure Proxmox

Now that we can use Ansible freely, we can use the `site.yml` playbook to set up most things. By default, no configuration changes will be applied unless the appropriate environment variable below is set to `true`. Many tasks support further configuration options, see the [README](./Ansible/roles/Configure_Proxmox/README.md) for the `Configure_Proxmox` role examples. 

| Variable                                 | Description                                                  | Example Value |
| ---------------------------------------- | ------------------------------------------------------------ | ------------- |
| `TKS_BP_T_CONFIGURE_REPOSITORIES`        | Use the [Contributor repositories](https://pve.proxmox.com/wiki/Package_Repositories) for packages | `true`        |
| `TKS_BP_T_CONFIGURE_UNATTENDED_UPGRADES` | Automatically manage [package updates](https://wiki.debian.org/UnattendedUpgrades) | `true`        |
| `TKS_BP_T_CONFIGURE_SYSTEM` | Configure system properties such as OS Swappiness | `true` |
| `TKS_BP_T_CONFIGURE_ZFS`              | Configures ZFS Memory Limitations, Swappiness, email notifications, etc. | `true`        |
| `TKS_BP_T_INSTALL_PACKAGES`              | Install a list of qualify-of-life packages for standard system administration | `true`        |
| `TKS_BP_T_INSTALL_SANOID`                | Install [Sanoid](https://github.com/jimsalterjrs/sanoid) and configure automatic ZFS Snapshot management | `true`        |
| `TKS_BP_T_INSTALL_POSTFIX` |Install and configure a [Postfix](http://www.postfix.org/) SMTP relay for email notifications|`true`|
| `TKS_BP_T_INSTALL_ZSH`                   | Install and configure [ZSH](https://www.zsh.org/) as the default user shell | `true`        |

For example, if you wanted to switch over to the contributor repositories, install my preferred qualify of life packages, configure ZFS, and set up unattended upgrades you might:

1. *Configure your Ansible client:*

   ```bash
   export ANSIBLE_REMOTE_USER="tj"
   export ANSIBLE_ASK_PASS=false
   export ANSIBLE_PRIVATE_KEY_FILE="~/.ssh/sol.milkyway"
   ```

2. *Export the variables indicating which configurations you wish to apply:* 

   ```bash
   export TKS_BP_T_CONFIGURE_REPOSITORIES=true
   export TKS_BP_T_INSTALL_PACKAGES=true
   export TKS_BP_T_INSTALL_POSTFIX=true 
   export TKS_BP_T_CONFIGURE_SYSTEM=true
   export TKS_BP_T_CONFIGURE_UNATTENDED_UPGRADES=true
   ```

3. *Define some variables to configure the `Postfix` relay client. Be mindful to not leave your password in your shell history:*

   ```bash
   export HISTCONTROL=ignoreboth
   export TKS_BP_V_POSTFIX_EMAIL="email@domain.com"
    export TKS_BP_V_POSTFIX_PASSWORD="YOURPASSWORD"
   export TKS_BP_V_POSTFIX_SERVER=smtp.gmail.com
   export TKS_BP_V_POSTFIX_PORT=587
   export TKS_BP_V_POSTFIX_TLS='yes'
   ```

4. *Configure the OS Swappiness as a percentage of 100.*

   ```bash
   export TKS_BP_V_SYS_SWAPPINESS=10
   ```
   
5. *Define some variables to configure Unattended Upgrades:*

   ```bash
   export TKS_BP_V_UPGRADES_NOTIFY=true
   export TKS_BP_V_UPGRADES_EMAIL="email@domain.com"
   export TKS_BP_V_UPGRADES_ON_SHUTDOWN=true
   export TKS_BP_V_UPGRADES_LOG_SYSLOG=true
   ```

6. *Apply the configurations to Proxmox:*

   ```bash
   ansible-playbook -i inventory.yml TKS-Bootstrap_Proxmox/Ansible/site.yml
   ```

   <hr>



### Configure Clustering

Your environment may or may not include multiple physical nodes. as a result, this step is **optional**. In order to form a cluster, you must have at least one master and node present in your inventory file under the groups `master` and `nodes`. Furthermore, only a single master can be in your inventory. Lastly, as a limitation invoked by Proxmox, your node can not have any workloads currently running on it.

Export the required environment variables and run the `configure_cluster.yml` Ansible Playbook. Be mindful to not leave your password in your shell history:

```bash
export HISTCONTROL=ignoreboth
export TKS_BP_V_PROXMOX_CLUSTER_NAME=TKS
 export TKS_BP_V_PROXMOX_MASTER_PASSWORD=YOURPASSWORD

ansible-playbook -i inventory.yml TKS-Bootstrap_Proxmox/Ansible/configure_cluster.yml
unset $TKS_BP_V_PROXMOX_MASTER_PASSWORD
```

Should something go wrong, or you wish you undo your clustering, you can do so with the following commands:

```bash
systemctl stop pve-cluster
systemctl stop corosync
pmxcfs -l
rm /etc/pve/corosync.conf 
rm /etc/corosync/*
killall pmxcfs
systemctl start pve-cluster
```

<hr>


### Configure Storage

Storage is a delicate component of any environment and there is a larger risk for disaster when applying automation to it as a result. Further complicating this, is that storage is configured differently in almost every environment. As a result, I have decided to leave this portion of TKS as a manual process. 

Our goal here is to provide our hypervisor with storage to use for three primary things. We'll need a place to store VMs, their backups, and general data. ZFS is a common storage solution for Proxmox, and can satisfy all three requirements in one go. My personal homelab leverages multiple ZFS and hardware arrays. For posterity, I'll include the steps I followed below. As a reminder, these commands will **NOT** be the same for your system.

1. *SSH into the server and configure LVM for the hardware RAID volume.*

   ```bash
   pvcreate /dev/sdg
   vgcreate RAIDPool /dev/sdg
   
   lvcreate RAIDPool /dev/sdg --name RAIDPool_Data -L 4T
   lvcreate RAIDPool /dev/sdg --name RAIDPool_Templates -L 100G
   lvcreate RAIDPool /dev/sdg --name RAIDPool_Backups -L 500G
   
   mkfs.xfs /dev/RAIDPool/RAIDPool_Data 
   mkfs.xfs /dev/RAIDPool/RAIDPool_Templates 
   mkfs.xfs /dev/RAIDPool/RAIDPool_Backups 
   ```
   
2. *Create working directories for Proxmox Backups, ISOs, & Templates and configure `/etc/fstab` accordingly.*

   ```bash
   mkdir /mnt/RAIDPool_Backups
   mkdir /mnt/RAIDPool_Templates
   mkdir /mnt/RAIDPool_Data
   
   echo "/dev/RAIDPool/RAIDPool_Templates /mnt/RAIDPool_Templates xfs defaults 0 0" >> /etc/fstab
   
   echo "/dev/RAIDPool/RAIDPool_Backups /mnt/RAIDPool_Backups xfs defaults 0 0" >> /etc/fstab
   
   echo "/dev/RAIDPool/RAIDPool_Data /mnt/RAIDPool_Data xfs defaults 0 0" >> /etc/fstab
   ```
   
3. *Import the ZFS Storage Pools.*

   ```bash
   zpool import -f DataPool
   zpool import -f FlashPool 
   # If necessary, use `zfs set` to change the mountpoints to `/mnt/` and re-import
   ```
   
4. *Make sure all of the filesystems have successfully mounted and then exit out of the SSH session.*

   ```bash
   zfs list
   zpool status
   
   mount -a
   df -h
   ```

5. *Within the Proxmox UI' s Datacenter `Storage` view, Disable the `local` Storage ID and remove the `local-lvm` Storage IDs to avoid over-provisioning the Proxmox OS SATADOM. Add the following Storage IDs:*

   | ID                 | Type      | Content                                             | Path                    | Shared | Enabled | Nodes | Thin | BS   |
   | ------------------ | --------- | --------------------------------------------------- | ----------------------- | ------ | ------- | ----- | ---- | ---- |
   | FlashPool          | ZFS       | Disk image, Container                               |                         | No     | Yes     | earth | No   | 32K  |
   | RAIDPool_Backups   | Directory | VZDump backup file                                  | /mnt/RAIDPool_Backups   | Yes    | Yes     | all   |      |      |
   | RAIDPool_Templates | Directory | Disk image, ISO image, Snippets, Container template | /mnt/RAIDPool_Templates | Yes    | Yes     | all   |      |      |
   | RAIDPool_Data      | LVM       | Disk image, Container                               |                         | No     | Yes     | earth | No   |      |
   | DataPool           | ZFS       | Disk image, Container                               |                         | No     | Yes     | earth | Yes  | 128K |

6. *Reboot the server and ensure that all of the storage endpoints are automatically mounted again.*

   ```bash
   reboot
   
   df -h
   ```

<hr>

