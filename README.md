
# TKS-Bootstrap_Proxmox

* [Summary](#Summary)
* [How To Use](#How-To-Use)
* [Instructions](#Instructions)
   * [Install Proxmox VE](#Install-Proxmox-VE)
   * [Create User Account](#Create-User-Account)
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

Now that Proxmox has been installed, it's time to set up a user account for Ansible to use. We'll also be creating an SSH key and adding it to the `authorized_keys` file for that user. 

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
   
   ansible-playbook -i inventory.ini TKS-Bootstrap_Proxmox/Ansible/create_user_account.yml
   ```

4. *Reconfigure Ansible to use SSH keys for authentication as well as your new user account.*

   ```bash
   export ANSIBLE_REMOTE_USER=tj
   export ANSIBLE_ASK_PASS=false
   export ANSIBLE_PRIVATE_KEY_FILE=~/.ssh/sol.milkyway
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
   
   # Set up SATADOM Proxmox Data partition as well.
   mkfs.xfs /dev/pve/data 
   ```

2. *Create working directories for Proxmox Backups, ISOs, & Templates and configure `/etc/fstab` accordingly.*

   ```bash
   mkdir /mnt/RAIDPool_Backups
   mkdir /mnt/RAIDPool_Templates
   mkdir /mnt/RAIDPool_Data
   
   echo "/dev/RAIDPool/RAIDPool_Templates /mnt/RAIDPool_Templates xfs defaults 0 0" >> /etc/fstab
   
   echo "/dev/RAIDPool/RAIDPool_Backups /mnt/RAIDPool_Backups xfs defaults 0 0" >> /etc/fstab
   
   echo "/dev/RAIDPool/RAIDPool_Data /mnt/RAIDPool_Data xfs defaults 0 0" >> /etc/fstab
   
   # Set up SATADOM Proxmox Data partition as well.
   mkdir /mnt/SATADOM_Data
   echo "/dev/pve/data /mnt/SATADOM_Data xfs defaults 0 0" >> /etc/fstab
   ```

3. *Import the ZFS Storage Pools.*

   ```bash
   zpool list
   zpool import -f DataPool
   zpool import -f FlashPool 
   # If necessary, use `zfs set` to change the mountpoints to `/mnt/` and re-import
   ```

4. *Make sure all of the filesystems have successfully mounted.*

   ```bash
   zfs list
   zpool status
   
   mount -a
   df -h
   ```

5. *Within the Proxmox Datacenter's `Storage` view, Disable the `local` Storage ID and remove the `local-lvm` Storage IDs to avoid over-provisioning the Proxmox OS SATADOM. Add the following Storage IDs:*

   | ID                 | Type      | Content                                             | Path                    | Shared | Enabled | Nodes | Thin | BS   |
   | ------------------ | --------- | --------------------------------------------------- | ----------------------- | ------ | ------- | ----- | ---- | ---- |
   | FlashPool          | ZFS       | Disk image, Container                               |                         | No     | Yes     | earth | No   | 32K  |
   | RAIDPool_Backups   | Directory | VZDump backup file                                  | /mnt/RAIDPool_Backups   | No     | Yes     | all   | No   |      |
   | RAIDPool_Templates | Directory | Disk image, ISO image, Snippets, Container template | /mnt/RAIDPool_Templates | Yes    | Yes     | all   | No   |      |
   | RAIDPool           | LVM       | Disk image, Container                               |                         | No     | Yes     | earth | No   |      |
   | DataPool           | ZFS       | Disk image, Container                               |                         | No     | Yes     | earth | Yes  | 128K |

   

<hr>

## Playbooks

| Playbook                | Description                                               |
| ----------------------- | --------------------------------------------------------- |
| create_usb_medium.yml   | Create a bootable usb drive containing the Proxmox VE ISO |
| create_user_account.yml | Create a user account and SSH key                         |

<hr>

## Roles

Most roles in TKS are configurable and, in some situations, entirely optional. Configuration of TKS is managed using environment variables. 

**Name:** [Create_USB_Medium](https://github.com/zimmertr/TKS-Bootstrap_Proxmox/tree/master/Ansible/roles/Create_USB_Medium)
**Description:** Create a bootable USB disk with a Proxmox VE ISO.

| Role | Description |
| ---- | ----------- |
|      |             |



<hr>

| Role                          | Description                                            | Environment Variable                     |
| ----------------------------- | ------------------------------------------------------ | ---------------------------------------- |
| Create_USB_Medium             | Create a bootable USB Flash Drive                      | N/A                                      |
| Create_User_Account           | Create an administrative user account and SSH key      | `TKS_BP_R_CREATE_USER_ACCOUNT`           |
| Configure_Proxmox             | Configures your Proxmox server                         | `TKS_BP_R_CONFIGURE_PROXMOX`             |
|                               |                                                        |                                          |
| Install_Packages              | Install qualify of life packages and update the system | `TKS_BP_R_INSTALL_PACKAGES`              |
| Install_ZSH                   | Install & configure ZSH                                | `TKS_BP_R_INSTALL_ZSH`                   |
| Install_Sanoid                | Install Sanoid for ZFS Snapshotting                    | `TKS_BP_R_INSTALL_SANOID`                |
| Install_Postfix               | Install Postfix as an SMTP Relay                       | `TKS_BP_R_INSTALL_POSTFIX`               |
| Configure_App_Armor           | Configure App Armor                                    | `TKS_BP_R_CONFIGURE_APP_ARMOR`           |
| Configure_ZFS                 | Tweak ZFS Memory Limits                                | `TKS_BP_R_CONFIGURE_ZFS`                 |
| Configure_Repositories        | Configure the package repositories                     | `TKS_BP_R_CONFIGURE_REPOSITORIES`        |
| Configure_ZED                 | Configure ZED for ZFS event alerting                   | `TKS_BP_R_CONFIGURE_ZED`                 |
| Configure_Unattended_Upgrades | Enable Proxmox to update its own packages              | `TKS_BP_R_CONFIGURE_UNATTENDED_UPGRADES` |
| Configure_Proxmox_Clustering  | Form a cluster with another Proxmox node               | `TKS_BP_R_CONFIGURE_PROXMOX_CLUSTERING`  |
| Configure_NTP_Servers         | Configure the upstream NTP Servers                     | `TKS_BP_R_CONFIGURE_NTP_SERVERS`         |

When necessary, use `export` to set a variable in your terminal before executing `ansible-playbook`. For example, if you only wanted to execute the role to install and configure ZSH:

```bash
export TKS_BP_R_INSTALL_ZSH=true
ansible-playbook -i inventory.yml site.yml
```

In many situations, a role may have optional or required configurable parameters that are also controlled via environment variables. They are documented in the individual `README.md` for each role which can be accessed by following the hyperlinks in the table below. When possible, default values are already provided for each role. However, this is not always possible. Variables for TKS are managed via environment variables instead of files for three reasons. 

1. The configuration hierarchy makes a lot more sense. The largest component of TKS is TKS itself -- the repository that aggregates everything. The smallest components are the Ansible roles and their individual tasks themselves. Default configuration values are therefore set at the bottom alongside the smallest components. And configuration overrides are performed at the highest level, alongside the biggest components. 
2. In some situations, variables are used for more than just Ansible. Using environment variables allows you to manage variables for all tooling in a standardized way.
3. Environment variables play better with CI/CD tooling. 
