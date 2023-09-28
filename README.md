# Bootstrap Proxmox

* [Summary](#Summary)
* [Instructions](#Instructions)
<hr>

## Summary

This Ansible project contains a few roles that apply common configuration changes to my personal Proxmox server. 

| Role                          | Description                                                  |
| ----------------------------- | ------------------------------------------------------------ |
| `configure_terraform_user`    | Create (or modify) a user, role, and API Token for use with the BPG Terraform Provider |
| `configure_zed`               | Configure ZED and install Systemd units to enable automatic zpool scrubbing |
| `install_base_packages`       | Install a handful of packages I typicall use on a base system |
| `install_postfix`             | Configure Postfix to send email notifications through `smtp.gmail.com` |
| `install_sanoid`              | Configure Sanoid to automatically create Zpool snapshots     |
| `isntall_unattended_upgrades` | Configure Unattended Upgrades to automatically install new packages |

<hr>

## Instructions

1. Configure any varibles as per your needs
2. Configure the inventory file according to your needs
3. `ansible-playbook -i inventory.yml prepare_proxmox.yml`
