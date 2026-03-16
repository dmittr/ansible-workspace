# Ansible Workspace Template

Infrastructure automation framework based on Ansible for managing baremetal and VPS servers.

## Overview

This project provides a unified approach to server management including:
- Automated package installation and user management
- Security configuration (SSH hardening, firewall)
- Support for both repository-based and custom binary installations
- Template-based custom installation scripts with override support

## Project Structure

```
code/
├── ansible.cfg              # Ansible configuration
├── inventory.yml            # Server inventory with host variables
├── requirements.yml         # External roles (e.g., wireguard)
├── site.yml                 # Main playbook (firewalld by default)
├── group_vars/
│   └── all/
│       └── vault.yml        # Encrypted secrets (Ansible Vault)
├── roles/                   # Custom roles
│   ├── docker-compose/      # Docker Compose deployment
│   ├── firewalld/           # Firewall management (core role)
│   ├── init-user-ssh/       # Initial server setup and SSH hardening
│   └── installer/           # Universal package/binary installer
├── override/                # Private templates (not in git)
│   └── installer/
│       └── templates/       # Custom installer script templates
└── external_roles/          # Roles from ansible-galaxy
```

## Roles

### 1. init-user-ssh

One-time role for initial server preparation:
- Creates admin user with SSH access
- Configures SSH security (custom port, key-only auth, no root)
- Sets up firewalld for SSH access
- Copies root's SSH keys to new user

**Usage:**
```bash
# Run once per new server (requires root access on port 22)
ansible-playbook roles/init-user-ssh/role.yml
```

### 2. firewalld (Core Role)

Manages firewall configuration:
- Interface to zone binding
- Port and service management
- Masquerade (NAT) configuration
- Port forwarding
- Rich rules for granular control

**Used in:** `site.yml`

### 3. installer

Universal installer supporting two modes:

#### Standard Installation (`install_packages`)
Repository-based package installation via dnf:
```yaml
install_packages:
  - name: docker-ce
    repo: "https://download.docker.com/linux/centos/docker-ce.repo"
    pkg:
      - docker-ce
      - docker-ce-cli
    service: docker
```

#### Custom Installation (`install_custom`)
Binary/script installations with idempotency check:
```yaml
install_custom:
  - name: node-exporter
    check_file: "/etc/systemd/system/node_exporter.service"
```

Custom installations use Jinja2 templates:
- Search order: `override/installer/templates/` → `roles/installer/templates/`
- Naming: `{name}_installer.sh.j2`
- Deployed to: `/opt/ansible-workspace-installer/{name}_installer.sh`

**Template Override:**
Place custom templates in `override/installer/templates/` to override role defaults without modifying the repository.

### 4. docker-compose

Deploys Docker Compose applications with:
- Data file synchronization
- Health checks (Docker native and HTTP)
- Pre-start command execution

## Quick Start

### 1. Clone and Setup

```bash
git clone <repository> ansible-workspace
cd ansible-workspace
```

### 2. Install External Roles

```bash
ansible-galaxy install -r requirements.yml -p ./external_roles
```

### 3. Configure Inventory

Edit `inventory.yml` with your server details:
```yaml
all:
  vars:
    ansible_ssh_private_key_file: "~/.ssh/id_ed25519"
    ansible_port: 33322
    firewalld_default_zone: "drop"
    # Change it. Better to use some folder outside of the repo
    override_path: "{{ playbook_dir }}/override"
  
  hosts:
    server.example.com:
      ansible_host: 192.168.1.2
      ansible_user: admin
      # Host-specific installer config
      install_packages:
        - name: docker-ce
          repo: "..."
          pkg: [docker-ce]
```

### 4. Setup Vault

```bash
# Create/edit vault
ansible-vault create group_vars/all/vault.yml

# Content:
vault_ansible_become_password: "your-sudo-password"
```

### 5. Initialize New Server

```bash
# First time only - requires root access
ansible-playbook roles/init-user-ssh/role.yml --limit server.example.com
```

### 6. Run Main Playbook

```bash
# Deploy firewall configuration
ansible-playbook site.yml
```

## Common Operations

### Update External Roles
```bash
ansible-galaxy install -r requirements.yml -p ./external_roles --force
```

### Edit Vault
```bash
ansible-vault edit group_vars/all/vault.yml
```

### Run Specific Role
```bash
ansible-playbook -e "install_packages=[{name: nginx,pkg: [nginx]}]" roles/installer/tasks/main.yml
```

## Installer Examples

### Standard: Install Nginx
```yaml
install_packages:
  - name: nginx
    repo: "https://nginx.org/packages/centos/nginx.repo"
    pkg: [nginx]
    service: nginx
```

### Custom: Install Node Exporter
```yaml
install_custom:
  - name: node-exporter
    check_file: "/usr/local/bin/node_exporter"
```

Template (`override/installer/templates/node-exporter_installer.sh.j2` or `roles/installer/templates/node-exporter_installer.sh.j2`):
```bash
#!/bin/bash
set -e
curl -LO https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
tar xzf node_exporter-*.tar.gz
cp node_exporter-*/node_exporter /usr/local/bin/
# ... create systemd service, etc.
```

## Security Features

- **Firewalld**: Default drop zone, explicit port allowlisting
- **SSH**: Non-standard port, key-only auth, no root login
- **Ansible Vault**: Encrypted secrets storage

## Network Configuration

Default zones:
- **drop**: Default deny-all zone
- **trusted**: Internal networks (docker0, VPN interfaces)
- **public**: Allowed external ports (80, 443, SSH)

## License

GNU General Public License v3.0
