# Ansible Workspace Template

Base template to your ansible workplace.

## Structure
- `roles/` — your custom roles.
- `external_roles/` — roles from ansible-galaxy.
- `group_vars/` — surprise, group vars.

## Quick start

1. Create `inventory.ini` from `example_inventory.ini` and add your servers
2. Copy `group_vars/example` to `group_vars/all` and provide needed values
3. Encrypt your vault `ansible-vault encrypt group_vars/all/vault.yml`
4. Edit `requirements.yml` with some essential roles you need and run `ansible-galaxy install -r requirements.yml -p ./external_roles`
5. If you have fresh machine with root access via ssh by key auth 

## Init-user-ssh

This is the first and one-shot role you need to prepare each server

1. Before use make sure you able to login as root with ssh-key on port 22.
2. run `ansible-playbook roles/init-user-ssh/role.yml` to create user, setup firewall and enforce ssh security.
