# Security Policy

## Supported Versions

There are no versions. `main` is what I run against my Proxmox server, and a fix ships as the next merge.

## Reporting a Vulnerability

Please report vulnerabilities privately through GitHub:
[Report a vulnerability](https://github.com/zimmertr/Bootstrap-Proxmox/security/advisories/new).
Do not open a public issue for security problems.

Include what you found, where it lives (role, file, or workflow), and how to reproduce it.

What to expect:

- **Acknowledgement** within a few days. This is a one-person project, so response times are best effort.
- **Fix**: a confirmed vulnerability is fixed in the next merge.
- **Credit** in the advisory, unless you'd rather not.

## Scope

In scope: the playbook, the roles and the CI workflows. A committed secret or a role that weakens the server's security is the likeliest finding.

Out of scope: Proxmox, Ansible and the packages the roles install. Report issues in those to their maintainers.

There is no bounty program. Reports are still appreciated.
