# Bootstrap-Proxmox

An Ansible playbook, `prepare_proxmox.yml`, that applies one role per concern to TJ's Proxmox server. The README lists the roles and how to run them.

This repo took a trimmed set of the files a new project starts with (zimmertr/yggdrasil record 0038). When a change here makes a decision, write its record; when it needs `CONTRIBUTING.md` or a `docs/` page, add it in the same PR.

## Rules

- Never run the playbook against the server without TJ's okay. Work in a fresh clone in the session's scratchpad, never in TJ's checkout, and never push to main.
- The README is TJ's: keep it accurate and in his voice, and never restructure it. A new role gets a row in its table.
- No secrets in the repo. `install_postfix` reads `SMTP_PASSWORD` from the environment.
- `install_fancontrol_tool` downloads from zimmertr/Supermicro-X10-Fancontrol's `main`.
- CI is zimmertr/yggdrasil's `pr-title` and `lint` modules, pinned by commit SHA with the version beside it. Renovate bumps the pin, and a patch or minor merges itself. The PR title is `type(scope): summary`.

## Issues

File issues with the `create-issue` skill from zimmertr/ai-config. The skill reads the four things below from this section.

- **The reader**: TJ, who runs the server and knows Proxmox and Ansible.
- **The gates**: `PR Title / PR Title` and the three `Lint /` checks pass; the README stays accurate.
- **The constraints**: never run the playbook or any command against the server without TJ's okay; no secret values in the repo or in logs.
- **The areas**: `ansible` (the playbook, roles, inventory), `ci` (the workflows), `security`.
