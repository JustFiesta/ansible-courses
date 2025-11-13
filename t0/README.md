# Ansible course - intro to Ansible

This folder contains excercises with Ansible as part of t0 learning.

There are my first steps and notes for Ansible.

Here I created the two tier application (webserver + lb) and fully configured it for sample PHP app.

---

## Infrastructure Setup for AWS

Upload setup-env.yml with changed to recent AMI ids, and your regions to CloudFormation

## Configure hosts

Update hosts-dev with Your own IP addresess and use it in configuration.

```shell
cd t0/

ansible-playbook -i hosts-dev playbooks/all-playbooks.yaml
```

---

## Notes on Ansible Basics

### Inventory check

```shell
ansible --list-hosts
```

NOTE: This also supports regular expressions and other types of calls, eg. `ansible --list-hosts app*`, `ansible --list-hosts webservers:loadbalancers`, `ansible --list-hosts \!control`, `ansible --list-hosts webservers[0]`

[More here](https://docs.ansible.com/ansible/latest/inventory_guide/intro_patterns.html)

---

### Ansible installation and configuration

Installation comes on UNIX machines with python via pip/pipx.

Configuration on nodes must provide generally accesible SSH key (`ssh-keygen`) and copied to hosts (`ssh-copy-id`). Additionally ansible user can be created on control and host node, with visudo -> `ansible_user ALL=(ALL) NOPASSWD: ALL` premission.

---

### Ansible tasks

* Basic building blocks of ansible execution.
* Oneliers for checking host status.
* They retrive return status of executed commands
* Tasks can be defined in playbooks

eg. `ansible -m ping all`

^ use module ping for inventory

---

### Playbooks

Recepie for certain host groups, ordered from first to last.

They are composed from plays (map of host-tasks). In plays modules are used (eg. copy, service, etc.)

eg. `ansible-playbook playbooks/ping.yml`

[Ansible playbooks](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html)

---

### Service handlers

Listeners for service config change. If change is made - service action is performed. Othervise service action is skipped

[Handlers](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_handlers.html)

---

### Compose multiple playbooks

Compose playbooks for one system in single playboook file. This happens via importing subsequent playbooks

eg.

```shell
  - import_playbook: check-status.yml
```

---

### Variables

Ansible provides ANSIBLE_FACTS variable with all metadata about host.  

Status module provides view into gathered information, eg. `ansible -m setup app1`  

Jinja2 templating is used to evaluate variables.

Local variables are created with `vars:`as key-value pairs.

Variables can be registered from tasks by `register: *name_of_variable*`. They will contain outputs from it's task

Debug module displays variables contents and playbook information.  

* [Using Variables](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html)
* [Register Variables](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#registering-variables)
* [Debug mode](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/debug_module.html)

---

### Roles

Used to group tasks together. Clean directory structure. Break configuration into files. Reuse code in similar configurations. Easy to modify and reduce syntax errors.

Initalize directory structure `ansible-galaxy role init roles/webservers`

It's a great way to write configuration in modular way.

[Roles](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html)
[Ansible Galaxy - repository with jump starts for projects](https://galaxy.ansible.com/ui/)

---

### Dry-run AKA Check mode

Dry-run does not make cnhanges on remote systems, rather than this it return status check if configuration could make any changes.

eg. `ansible-playbook playbooks/setup-webapp --check`

---

### Error handling in playbooks

Some modules might return non zero exit code and its okay (eg. `command: /basg/false`), or they might return change status when change was NOT made (eg. `command: service httpd status`)

In this scenarios one can use `ignore_errors: yes` and `changed_when: false`

[Error handling](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_error_handling.htmls)

---

### Tags

Add them to specific tasks, so one can use specyfic parts of playbook.

Can be added to any resource.

Specify tags one want to run.

eg. `ansible-playbook playbooks/setup-webapp.yml --tags upload`

or skip specyfic tasks:

eg. `ansible-playbook playbooks/setup-webapp.yml --skip-tags upload`

[Tags](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_tags.html)

---

### Ansible Vault - keep sensitive data safe

Encrypts stored vault files. Vault can be shared via SCM. Vault can encrypt any data structure used by Ansible. Password protected. Default cipher is AES.

* create vault: `ansible-vault create vars/secret-variables.yml`
* edit vault file: `ansible-vault edit vars/secret-variables.yml`
* prompt for password: `ansible-playbook setup-app.yml --ask-vault-pass`

[Ansible Vault](https://docs.ansible.com/ansible/latest/vault_guide/index.html)

---

### Prompts

Can be stored as ariables, run conditions based on them, ask for sensitive data.

To ask user a prompt, in a playbook file specify:

```bash
vars_prompt: 
    - name: "upload_var"
      prompt: "Upload index.php file?"

tasks:
    - name: Upload application file
      copy:
        src: ../index.php
        dest: "{{ path_to_app }}"
      when: upload_vars == 'yes'
      tags: upload
```

Above yml has a conditional based on user input in upload task (`when: ...`). When input is 'yes' it will proceed with task, otherwise it is omited.

[Prompts](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_prompts.html)
[When Statement](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_conditionals.html#the-when-statement)
