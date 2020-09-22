Configure Proxmox
=========

Configure a newly installed Proxmox VE server.

Tasks
-----

| Task                                | Description                                                  |
| ----------------------------------- | ------------------------------------------------------------ |
| `configure_repository_contrib.yml`  | Configure apt to use the Proxmox contributor repositories.   |
| `configure_unattended_upgrades.yml` | Installs and configures Unattended Upgrades.                 |
| `configure_zfs.yml`                 | Configures ZFS Memory Limitations, Swappiness, email notifications, etc. |
| `install_base_packages.yml`         | Installs and configures my desired base packages.            |
| `install_postfix_relay.yml`         | Installs and configures a Postfix relay.                     |
| `main.yml`                          | The main task that is executed automatically by the role. Calls other tasks depending on desired state. |



Role Variables
--------------

| Variable                             | Description                                               | Has Default | Example               |
| ------------------------------------ | --------------------------------------------------------- | ----------- | --------------------- |
| `TKS_BP_V_POSTFIX_SERVER`            | Remote SMTP Server                                        | Yes         | `smtp.gmail.com`      |
| `TKS_BP_V_POSTFIX_PORT`              | Remote SMTP Port                                          | Yes         | `587`                 |
| `TKS_BP_V_POSTFIX_TLS`               | Remote SMTP Server TLS Support                            | Yes         | `yes`                 |
| `TKS_BP_V_POSTFIX_EMAIL`             | Remote SMTP Server Username                               | No          | `email@gmail.com`     |
| `TKS_BP_V_POSTFIX_PASSWORD`          | Remote SMTP Server Password                               | No          | `PASSWORD`            |
| `TKS_BP_V_UPGRADES_NOTIFY`           | Enable notifications for unattended upgrades              | Yes         | `true`                |
| `TKS_BP_V_UPGRADES_NOTIFY_ON_ERROR`  | Only send notifications for upgrades on error             | Yes         | `false`               |
| `TKS_BP_V_UPGRADES_EMAIL`            | The destination email address for unattended upgrades     | No          | `email@gmail.com`     |
| `TKS_BP_V_UPGRADES_UNUSED_DEP`       | Automatically remove unused dependencies                  | Yes         | `false`               |
| `TKS_BP_V_UPGRADES_UNUSED_K_PAC`     | Automatically remove unused kernel packages               | Yes         | `false`               |
| `TKS_BP_V_UPGRADES_AUTO_REBOOT`      | Automatically reboot the node to install package upgrades | Yes         | `false`               |
| `TKS_BP_V_UPGRADES_AUTO_REBOOT_TIME` | Schedule a time when automatic reboots should occur       | Yes         | `03:00`               |
| `TKS_BP_V_UPGRADES_ON_SHUTDOWN`      | Only install upgrades when a graceful shutdown occurs     | Yes         | `true`                |
| `TKS_BP_V_UPGRADES_LOG_SYSLOG`       | Log all upgrade messages to the SYSLOG                    | Yes         | `true`                |
| `TKS_BP_V_SYS_SWAPPINESS`            | The level of Swappiness applied to the OS                 | Yes         | `10`                  |
| `TKS_BP_V_ZFS_POOL_LIST`             | Enable timed scrubbing on ZFS pools, comma separated      | No          | `DataPool,FlashPool`  |
| `TKS_BP_V_SANOID_VERSION`            | The version of Sanoid to install.                         | Yes         | `2.0.3`               |


License
-------

GPL-3.0-only

Author Information
------------------

| Contact  | Information                           |
| -------- | ------------------------------------- |
| Name     | TJ Zimmerman                          |
| Email    | tj@tjzimmerman.com                    |
| Website  | https://tjzimmerman.com               |
| GitHub   | https://github.com/zimmertr           |
| LinkedIn | https://www.linkedin.com/in/zimmertr/ |
| Twitter  | https://twitter.com/zimmertr          |

