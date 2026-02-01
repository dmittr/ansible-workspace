# Ansible Workspace Template

Base template to your ansible workplace.

## Structure
- `roles/` — your custom roles.
- `external_roles/` — roles from ansible-galaxy.
- `group_vars/` — surprise, group vars.

## Quick start

1. Create a project folder and clone this repo `git clone git@github.com:dmittr/ansible-workspace.git`
2. Copy files to your project `cp -r ansible-workspace/{group_vars,ansible.cfg,inventory.yml,site.yml} .`
3. Update external_roles `cd ansible-workspace && ansible-galaxy install -r requirements.yml -p ./external_roles`
4. Populate `inventory.yml` in your project folder with your own data.
5. Encrypt your vault `ansible-vault encrypt group_vars/all/vault.yml`

## Init-user-ssh

This is the first and one-shot role you need to prepare each server

1. Before use make sure you able to login as root with ssh-key on port 22.
2. run `ansible-playbook roles/init-user-ssh/role.yml` to create user, setup firewall and enforce ssh security.
