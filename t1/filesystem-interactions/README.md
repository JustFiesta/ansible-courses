# File System Manipulations

Ansible provides numerous vays to manipulate file system. Eg. Users can create isos, create links, set permissions, edit file, and many more.

In this folder are some advances examples actions user can Perform.

## Basic Files operations

Core file system operations for managing files and directories on remote hosts

### File Management

Create, delete, and manage file/directory properties.

```yaml
- name: Create empty file
  ansible.builtin.file:
    path: /tmp/example.txt
    state: touch

- name: Create directory
  ansible.builtin.file:
    path: /opt/myapp
    state: directory
    mode: '0755'

- name: Set file permissions
  ansible.builtin.file:
    path: /etc/config.cfg
    owner: root
    group: wheel
    mode: '0644'

- name: Rename file
  ansible.builtin.command:
    cmd: mv /old/name.txt /new/name.txt
```

### Linking

Create symbolic and hard links.

```yaml
- name: Create symbolic link
  ansible.builtin.file:
    src: /original/file
    dest: /link/to/file
    state: link

- name: Create hard link
  ansible.builtin.file:
    src: /original/file
    dest: /hardlink/to/file
    state: hard
```

### Find operations

Rename and find files by extension.

```yaml
- name: Find all .log files
  ansible.builtin.find:
    paths: /var/log
    patterns: '*.log'
    recurse: yes
  register: log_files

- name: Delete files in directory
  ansible.builtin.find:
    paths: /tmp/cache
    age: 7d
  register: old_files

- name: Remove found files
  ansible.builtin.file:
    path: "{{ item.path }}"
    state: absent
  loop: "{{ old_files.files }}"
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

## File Edits

Ansible builtin provides two ways to edit files. Single and multi-line edits. They support regex and other normal structure (state, dest, etc.)

### Single Line

[Module ref](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/lineinfile_module.html)

### Multi Line block

[Module ref](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/blockinfile_module.html)

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

## Archive operations

Create and extract archive files on remote hosts.

```yaml
- name: Create compressed archive
  ansible.builtin.archive:
    path: /var/log/
    dest: /tmp/logs.tar.gz
    format: gz

- name: Extract archive
  ansible.builtin.unarchive:
    src: /tmp/software.tar.gz
    dest: /opt/software
    remote_src: yes
```

Reference: [Archive](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/archive_module.html) | [Unarchive](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/unarchive_module.html)

## Create ISO

Create ISO images from directories or files.

```yaml
- name: Create bootable ISO
  community.general.iso_create:
    src: /path/to/source/
    dest: /tmp/image.iso
    volume_id: "MY_ISO"
    rockridge: yes
    joliet: yes
```

[Module ref](https://docs.ansible.com/projects/ansible/latest/collections/community/general/iso_create_module.html)

## File Reading

Read and display file contents from remote hosts.

```yaml
- name: Read configuration file
  ansible.builtin.slurp:
    src: /etc/app/config.conf
  register: config_content

- name: Display file content
  ansible.builtin.debug:
    msg: "{{ config_content.content | b64decode }}"

- name: Read first lines of log
  ansible.builtin.shell:
    cmd: head -20 /var/log/app.log
  register: log_head

- name: Show log excerpt
  ansible.builtin.debug:
    var: log_head.stdout
```

## File Validation

Validate file syntax and integrity.

```yaml
- name: Validate JSON file
  ansible.builtin.shell:
    cmd: python -m json.tool /etc/app/config.json
  register: json_validation
  failed_when: json_validation.rc != 0

- name: Validate YAML file
  ansible.builtin.shell:
    cmd: python -c "import yaml; yaml.safe_load(open('/etc/app/config.yaml'))"
  register: yaml_validation

- name: Check configuration syntax
  ansible.builtin.command:
    cmd: nginx -t -c /etc/nginx/nginx.conf
  register: nginx_check
  changed_when: false
```

## Mount NFS

Instead of running manual configurations Users can use provided modules that take heavy lifting from them.

### Linux NFS

Mount NFS shares on Linux systems using the posix mount module.

```yaml
- name: Mount NFS share
  ansible.posix.mount:
    path: /mnt/nfs_share
    src: '192.168.1.100:/export/data'
    fstype: nfs
    opts: defaults
    state: mounted
  become: true
```

### Windows Network Drives

Map network drives on Windows systems.

```yaml
- name: Map network drive
  community.windows.win_mapped_drive:
    letter: Z
    path: \\server\share
    state: present
```

[Module ref](https://docs.ansible.com/projects/ansible/latest/collections/community/windows/win_mapped_drive_module.html)

## Synchronize

Efficient file synchronization using rsync algorithm. Can be used for many different files/backups/logs/etc.

```yaml
- name: Sync directories
  ansible.posix.synchronize:
    src: /local/path/
    dest: /remote/path/
    recursive: yes
    delete: yes
```
