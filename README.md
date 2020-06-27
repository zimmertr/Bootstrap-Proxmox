# TKS-Bootstrap_Proxmox

This repository is a submodule of TKS. It can be used on its own, but is intended to be used as a component of a larger ecosystem. It is recommended that you instead clone the parent repository and use this submodule from there.

## Summary

`Bootstrap_Proxmox` is a collection of Ansible roles used to prepare a newly installed Proxmox server for TKS. The included roles are documented below. 

## Roles

All of the required roles are automatically included in the `site.yml` playbook according to whether or not specific environment variables are set to `true` at runtime. Some roles and scripts are not meant to be run with Ansible, have unique instructions, and do not require the use of environment variables. They are indicated accordingly.

| Role                          | Description                                            | Environment Variable                   |
| ----------------------------- | ------------------------------------------------------ | -------------------------------------- |
| Create_USB_Medium             | Create a bootable USB Flash Drive                      | N/A                                    |
| Create_SSH_Key                | Generate a new SSH Key                                 | N/A                                    |
| Create_User_Account           | Create an administrative user account                  | `TKS_BP_R_CREATE_USER_ACCOUNT`           |
| Install_SSH_Key               | Copy an SSH Key to the Proxmox server                  | N/A                                    |
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

## Instructions
