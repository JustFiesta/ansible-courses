# Ansible T1 Assessment Project

This project demonstrates infrastructure automation skills using Ansible to deploy and configure a complete web application environment with load balancing. The setup includes multiple web servers behind an HAProxy load balancer, with environment-specific configurations and security measures.

## What This Project Demonstrates

- Configuration as Code: Complete environment provisioning through Ansible playbooks
- Multi-tier Architecture: Web servers + load balancer configuration

## Prerequisites

Below tools needed to be present on local machine:

- Ansible

```shell
pipx install ansible
```

- Docker (i used [Docker Desktop](https://docs.docker.com/desktop/) + WSL)

Here are docs how to install [CLI docker](https://docs.docker.com/engine/install/debian/)

- sshpass

```bash
# tool to connect to Docker hosts with password for demonstration
sudo apt install sshpass
```

## Project Setup

Here are steps reqiured to succesfully t1 project.

The end goal is to see "Welcome!" on `localhost` GET (curl or browser).

### Reqiurements install

t1 project requires external collection for managing MySQL database. It needs to be downloaded before using ansible to setup infrastructure.

```shell
ansible-galaxy install -r requirements.yml
```

### Configure Ansible Vault

1. Edit the Vault variables file at group_vars/all/vault.yaml:

    ```yaml
    mysql_root_password: "YourNewPassword"
    mysql_db_name: "mydb"
    ```

2. Encrypt the file if it’s not already encrypted:

    ```shell
    ansible-vault encrypt group_vars/all/vault.yaml
    ```

### Run Docker infrastructure

Fire up guest systems.

```shell
cd t1/docker/

docker compose up -d
```

To disable infrastructure run:

```shell
cd t1/docker/

docker compose down -v
```

### Run Ansible configuration with Vault

Since Vault is setted up user needs to add `--ask-vault-pass` into `ansible-playbook` command.

```shell
cd t1/

ansible-playbook playbooks/full-app-stack-setup.yaml -i inventories/hosts.yaml --ask-vault-pass
```

or store password to Vault in some textfile:

```shell
ansible-playbook playbooks/full-app-stack-setup.yaml -i inventories/hosts.yaml --vault-password-file ~/.vault_pass.txt
```

### Verification

HAProxy stats page: `http://localhost:8404/stats`.

Also on `http://localhost:80` should display "Welcome!".

## Steps taken to create project

Figured out sample app from hands-on course and created all files.

Role boilerplate were created with `ansible-galaxy init`, later I removed not used files.

My approach to creation of Ansible configuration was to:

- figure out needed infrastructure
- set it up with docker
- test connection to guests with module ping
- create main playbook and sketch its includes
- create roles boilerplate
- go role by role and add configuration while reading docs in browser on the modules/collections examples
- to speed up I used LLMS to generate role with detailed prompts

### Encoutered problems

The only one was with database configuration.

At first I could not figure out socket authentication for root. But after some prompts and reading docs I figured what arguments Ansible needs for correct configuration.
