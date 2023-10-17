# Bootstrap Proxmox

* [Summary](#Summary)
* [Instructions](#Instructions)
<hr>

## Summary

This Ansible project contains a few roles that apply common configuration changes to my personal Proxmox server. 

| Role                          | Description                                                  |
| ----------------------------- | ------------------------------------------------------------ |
| `configure_cluster`           | Create a single-node Proxmox cluster                         |
| `configure_zed`               | Configure ZED and install Systemd units to enable automatic zpool scrubbing |
| `create_user`                 | Create (or modify) a user, role, and API Token               |
| `enable_iommu`                | Configure GRUB and enable the kernel modules required for enabling IOMMU |
| `install_base_packages`       | Install a handful of packages I typically use on a base system |
| `install_nfs_server`          | Install NFS Kernel Server and Configure `/etc/exports`       |
| `install_postfix`             | Configure Postfix to send email notifications through `smtp.gmail.com` |
| `install_sanoid`              | Configure Sanoid to automatically create Zpool snapshots     |
| `install_unattended_upgrades` | Configure Unattended Upgrades to automatically install new packages |

<hr>

## Instructions

1. Configure any varibles as per your needs.
2. Configure the inventory file according to your needs.
3. Run the Ansible Playbook: `ansible-playbook -i inventory.yml prepare_proxmox.yml`
   * If you only want to run a specific role, pass in the name of the role as a tag. For example: `--tags create_user`
