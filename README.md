# Ansible Learning Repository

This repository documents my learning journey with Ansible across different skill levels and use cases. Each directory contains standalone course I've completed.

Assesment are the "final" projects for testing knowlage and showcase usecase.

## What is Ansible?

* open-source automation tool for configuration management, application deployment, and task orchestration.
* uses SSH for communication
* no agents on managed nodes required.

## Installation

### Prerequisites

* Python 3.8 or newer
* SSH access to managed nodes
* Control node running Linux, macOS, or WSL

### Installing Ansible

**pipx for global isolation:**

```bash
pipx install ansible
```

**Verify installation:**

```bash
ansible --version
```

## Initial Setup

### SSH Configuration

Generate SSH key pair on control node:
```bash
ssh-keygen -t rsa -b 4096
```

Copy public key to managed hosts:
```bash
ssh-copy-id user@hostname
```

### Basic Configuration File

Create `ansible.cfg` in your project directory:
```ini
[defaults]
inventory = ./hosts
remote_user = ansible
private_key_file = ~/.ssh/id_rsa
host_key_checking = False
```

### Inventory File

Create basic inventory file (`hosts`):

```ini
[webservers]
web1 ansible_host=192.168.1.10
web2 ansible_host=192.168.1.11

[databases]
db1 ansible_host=192.168.1.20

[all:vars]
ansible_user=ansible
```

## Basic Commands

### Ad-hoc Commands

Test connectivity:

```bash
ansible all -m ping
```

Run shell command:

```bash
ansible webservers -m shell -a "uptime"
```

Install package:

```bash
ansible all -m package -a "name=vim state=present" --become
```

### Inventory Management

List all hosts:

```bash
ansible --list-hosts all
```

List specific group:

```bash
ansible --list-hosts webservers
```

Pattern matching:

```bash
ansible --list-hosts "web*"
ansible --list-hosts "webservers:databases"
ansible --list-hosts "!control"
```

### Playbook Execution

Run playbook:

```bash
ansible-playbook playbook.yml
```

Check mode (dry-run):

```bash
ansible-playbook playbook.yml --check
```

With specific tags:

```bash
ansible-playbook playbook.yml --tags "configuration"
```

Skip specific tags:

```bash
ansible-playbook playbook.yml --skip-tags "cleanup"
```

Verbose output:

```bash
ansible-playbook playbook.yml -v
ansible-playbook playbook.yml -vvv  # More verbose
```

## Core Concepts Reference

### Modules

Reusable units of work. Common modules include:

* `ping` - Test connectivity
* `copy` - Copy files to remote hosts
* `template` - Template files using Jinja2
* `package` - Manage packages

### Playbooks

YAML files defining automation tasks. Basic structure:

```yaml
---
- hosts: webservers
  become: true
  tasks:
    - name: Install nginx
      package:
        name: nginx
        state: present
```

### Variables

Access variables using Jinja2 syntax:

```yaml
{{ variable_name }}
{{ ansible_facts['hostname'] }}
```

Gather facts:

```bash
ansible hostname -m setup
```

### Roles

Organized directory structure for reusable content:

```bash
ansible-galaxy role init role_name
```

Standard role structure:

```shell
role_name/
├── defaults/
├── files/
├── handlers/
├── tasks/
├── templates/
├── vars/
└── meta/
```

### Handlers

Tasks triggered by notifications, typically for service restarts:

```yaml
handlers:
  - name: restart nginx
    service:
      name: nginx
      state: restarted
```

## Docs

* [Official Documentation](https://docs.ansible.com/)
* [Ansible Galaxy](https://galaxy.ansible.com/) - Community roles and collections
* [Best Practices](https://docs.ansible.com/ansible/latest/tips_tricks/ansible_tips_tricks.html)
