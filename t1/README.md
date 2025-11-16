# More advanced Ansible usage

Here are excercises I performed to get more knowlage of Ansible

## Dry running with diff

Shows changes that Ansible will perform without propagating them to hosts.

```shell
ansible-playbook playbooks/some-playbook.yasml --check --diff
```

## Ansible AWX

The UI for Ansible installation - serves as Full Automation platform. Runs as container (eg. in k8s)

It is an open-source version of Red Hat AAP/Tower

It has:

- web UI
- AWX REST API
- Task engine - for task/playbook queues. Separeted from environment with containers.
- PostgreSQL - config and history db
- Redis - queue of tasks

User has options to define:

- project - git repos with playbooks
- inventories
- credentials
- templates - ready to run playbooks with params
- schedules/worfklows - auto run / contatenation of many tasks

Can be used as Audit/Automation Center. Also integrates with CI/CD.

## Custom Standalone Ansible Runners

Sometimes ansible Runners need to have additional python libraries; ansible collections or other binaries. With `ansible-builder` user can create standalone container with all dependencies and use it as Ansible runner.

This tool enables users to create stable and repetetive Ansible runner environments.

AWX and AAP are using EE.

### Needed to build Execution Environment

- `execution-environment.yaml` - schema for building EE
- Ansible Collection (for installing in EE)
- Python libraries (requirements.txt or package names)
- binaries (bindep.txt)

eg.

```shell
version: 3
dependencies:
  python: requirements.txt
  system: bindep.txt
  galaxy: requirements.yml

additional_build_step:
  prepend: |
    RUN pip3 install --upgrade pip setuptools
  append:
    - RUN ls -la
```

### Building new EE

It will create ready to use container with Ansible and dependencies stated in `execution-environment.yml`

```shell
ansible-builder build -t my-ee:latest
```

### Use new EE

```shell
ansible-runner run -p playbooks/test-playbook.yaml --inventory inventories/my-inventory.yaml --container-image=my-ee
```

## Ansible Config

In ansible.cfg there are many options to tailor Ansible actions.

One of them is `host_key_checking`. This might speed up process when set to False.

(Other Configuration Options)[https://docs.ansible.com/projects/ansible/latest/reference_appendices/config.html]

Also user can debug current configuration with `ansible-config` tool.

```shell
ansible-config dump   # show current config whole
ansible-config dump --only-changed  # show only changed values in config (stated by user)
```

## Special Ansible variables

Ansible provides reserved variables. Magic variables are special built-in variables automatically available in your playbooks, roles, and tasks. You don’t need to define them, and they provide information about the current play, host, inventory, or task execution.

[Magic Vars ref](https://docs.ansible.com/projects/ansible/latest/reference_appendices/special_variables.html)

## Run Handlers only on change

On default Ansible Runs all Handlers after all tasks. Notified handlers are executed automatically after each of the following sections

To change that use flush before handler usage:

```yaml
tasks:
  - name: Some tasks go here
    ansible.builtin.shell: ...

  - name: Flush handlers
    meta: flush_handlers

  - name: Some other tasks
    ansible.builtin.shell: ...
```

[More on Handlers](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_handlers.html)

## Handlers listeners

Handlers can listen on topics that can group multiple handlers as follows:

```yaml
tasks:
  - name: Restart everything
    command: echo "this task will restart the web services"
    notify: "restart web services"

handlers:
  - name: Restart memcached
    service:
      name: memcached
      state: restarted
    listen: "restart web services"

  - name: Restart apache
    service:
      name: apache
      state: restarted
    listen: "restart web services"
```

## Ansible Vault

Tool for secret management (password, files, etc.). Included in base installation of Ansible. The `ansible-vault` CLI is used to manage Vault and Secrets.

In one Ansible system there might be many Vaults.

TIP: Store passwords for files in Password Manager as Bitwarden.

eg. File:

```yaml
---
password: SuperSecretPass123
```

### How-to encrypt

```shell
ansible-vault encrypt password.yaml

# Place password for file encryption
```

### How-to decrypt

```shell
ansible-vault decrypt password.yaml

# Place password for file encryption
```

## Ansible Vault in Playbook

1. Place variable in separete YAML file
2. Encrypt with Vault
3. Add include vault task:

    ```shell
    - name: Include Vault Var
    ansible.builtin.include_vars:
        file: password.yaml
    ```

4. Run playbook with `--ask-vault-password`

## Ansible builtin vs Ansible Legacy modules

- `ansible.builtin` - the collection shipped with `ansible-core`. Basic modules splitted into collections
- `ansible.legacy` - superset of `ansible.builtin`, with custom plugins in configured paths. Before Collection style modules were supported. Left as compatibility layer.

Instead of using legacy mappings:

```yaml
- name: Create catalog
  file:
    path: /tmp/test
    state: directory
```

Now it is required to specify used module:

```yaml
- name: Create catalog
  ansible.builtin.file:
    path: /tmp/test
    state: directory
```

## Ansible builtin

All builtin collections are present for numerous of common tasks (reboot, shutdown, ping, file edits, service management, etc.)

[Collection ref](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/index.html)

## File edits

Ansible builtin provides two ways to edit files. Single and multi-line edits. They support regex and other normal structure (state, dest, etc.)

### Single Line

[Module ref](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/lineinfile_module.html)

### Multi Line block

[Module ref](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/blockinfile_module.html)

## Checkout Git Repository on Target Hosts

Git module enables users to deploy software from git repositories (HTTPS/SSH connection types).

[Module ref](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/git_module.html#ansible-collections-ansible-builtin-git-module)

### SSH connection Checkout

This requires SSH key present on target machine so it can connect to repo.

```yaml
- name: git module demo
  hosts: all
  vars:
    repo: "git@github.com:user/repo.git"
    dest: "/home/ansible_user/repo"
    sshkey: "~/.ssh/id_rsa"
  tasks:
    - name: ensure git pkg installed
      ansible.builtin.yum:
        name: git
        state: present
        update_cache: true
      become: true

    - name: checkout git repo
      ansible.builtin.git:
        repo: "{{ repo }}"
        dest: "{{ dest }}"
        key_file: "{{ sshkey }}"
```

### HTTPS connection Checkout

```yaml
- name: git module demo
  hosts: all
  tasks:
    - name: ensure git pkg installed
      ansible.builtin.yum:
        name: git
        state: present
      become: true

    - name: checkout git repo
      ansible.builtin.git:
        repo: https://github.com/user/repo.git
        dest: /home/ansible_user/repo
```

## Copy files

`fetch` and `copy` modules enable users to fetch files from hosts machines.

### Remote -> local

```yaml
- name: fetch module demo
  hosts: all
  become: true
  vars:
    log_file: "/var/log/messages"
    dump_dir: "logs"
  tasks:
    - name: fetch log
      ansible.builtin.fetch:
        src: "{{ log_file }}"
        dest: "{{ dump_dir }}"
```

[fetch ref](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/fetch_module.html#ansible-collections-ansible-builtin-fetch-module)

### Local -> Remote

```yaml
- name: copy module demo
  hosts: all
  become: false
  tasks:
    - name: copy report.txt
      ansible.builtin.copy:
        src: report.txt
        dest: /home/ansible_user/report.txt
        owner: ansible_user
        mode: '0644'

```

[copy ref](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/copy_module.html#ansible-collections-ansible-builtin-copy-module)

## Read files from remote host

`ansible.builtin.slurp` Reads a file from a remote host and returns its content base64-encoded.

Useful for fetching configuration files, certificates, or any file for auditing or processing in playbooks.

TIP: Remember it requires base64 (ENCODE and DECODE) for usage of file contents.

```yaml
- name: Slurp remote file
    ansible.builtin.slurp:
    src: /etc/hostname
    register: hostname_raw

- name: Decode and show file content
    ansible.builtin.debug:
    msg: "{{ hostname_raw.content | b64decode }}"
```

## Templating

Some files might be very similar or just differ in few places (eg. variables, hosts, IP, etc.). Templating comes in handy here.

Create Jinja2 template file and populate it with cusomized variabled via task.

### Example Nginx HTML template

```j2
<html>
<head>
  <title>{{ page_title }}</title>
</head>
<body>
    <h1>{{ page_title }}</h1>
    <p>{{ page_description }}</p>
</body>
</html>
```

### Template usage

```yaml
- name: template module demo
  hosts: all
  become: true
  vars:
    page_title: "Placeholder"
    page_description: |
      This is my placeholder page example.
      Multiline is possible ;-)
  tasks:
    - name: install Nginx
      ansible.builtin.apt:
        name: nginx
        state: latest

    - name: apply page template
      ansible.builtin.template:
        src: templates/placeholder.html.j2
        dest: /var/www/html/index.html
```

### Loop in file template

Very usefull for any configuration file with repeating patterns.

File template:

```j2
{% for site in sites %}
server {
    listen 80;
    server_name {{ site.server_name }};

    root {{ site.root }};
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}
{% endfor %}
```

Playbook usage:

```yaml
- name: Render Nginx virtual hosts
  ansible.builtin.template:
    src: nginx_vhosts.j2
    dest: /etc/nginx/conf.d/vhosts.conf
  vars:
    sites:
      - server_name: example.com
        root: /var/www/example
      - server_name: test.com
        root: /var/www/test
```

## Host service management

Ansible provides two modules for service management:

- `ansible.builtin.service` - service state management (has many parameters to tailor usecase, eg. enable, restart, start, etc.)
- `ansible.windows.win_service` - additional windows service state management
- `ansible.builtin.service_facts` - service state information

eg.

```yaml
- name: service module demo
  hosts: all
  become: true
  vars:
    disable_services:
      - "nginx.service"
  tasks:
    - name: Get Service Info
      ansible.builtin.service_facts:

    - name: Disable services
      ansible.builtin.service:
        name: "{{ item }}"
        enabled: false
        state: stopped
      when: "item in services"
      with_items: '{{ disable_services }}'

    - name: Check Nginx state
      debug:
        msg: "{{ ansible_facts.services['nginx.service'].  state }}"
```

[service ref](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/service_module.html#ansible-collections-ansible-builtin-service-module)
[service_facts ref](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/service_facts_module.html#ansible-collections-ansible-builtin-service-facts-module)

## CRON job schedule

Ansible provides builtin controller for cron.d. It enables users to control CRONs as a Code.

### Add cronjob

```yaml
- name: sample cronjob add
  ansible.builtin.cron:
    name: "backup /tmp"
    minute: "0"
    hour: "2"
    job: "/usr/bin/rsync -a /tmp /backup"
```

### Remove cronjob

```yaml
- name: Remove cronjob
  ansible.builtin.cron:
    name: "backup /tmp"
    state: absent
```

[cron ref](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/cron_module.html#ansible-collections-ansible-builtin-cron-module)
