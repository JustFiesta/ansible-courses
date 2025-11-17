# User and Group Management

Manage system users and groups including account lifecycle, authentication, and group membership. It is very good way to create service users.

## User Management

Create, Update, and Delete Users

```yaml
- name: Create new user
  ansible.builtin.user:
    name: johndoe
    comment: "John Doe"
    groups: developers,users
    shell: /bin/bash
    state: present

- name: Update user properties
  ansible.builtin.user:
    name: johndoe
    comment: "John Doe - Developer"
    append: yes
    groups: docker

- name: Remove user
  ansible.builtin.user:
    name: johndoe
    state: absent
    remove: yes
```

## Account Status Management

Lock, enable and disable accounts. Can be parametrized and used as adhoc/group control in company.

```yaml
- name: Disable user account
  ansible.builtin.user:
    name: johndoe
    state: present
    shell: /sbin/nologin

- name: Lock user account
  ansible.builtin.user:
    name: johndoe
    state: present
    password: '!'

- name: Enable user account
  ansible.builtin.user:
    name: johndoe
    state: present
    shell: /bin/bash
```

## Password Management

```yaml
- name: Set user password
  ansible.builtin.user:
    name: johndoe
    password: "{{ 'password123' | password_hash('sha512') }}"

- name: Force password change on login
  ansible.builtin.user:
    name: johndoe
    update_password: always
```

## Group Management

Group CRUD Operations

```yaml
- name: Create group
  ansible.builtin.group:
    name: developers
    state: present
    gid: 2000

- name: Modify group
  ansible.builtin.group:
    name: developers
    gid: 2001
    state: present

- name: Remove group
  ansible.builtin.group:
    name: developers
    state: absent
```

## Group Membership Management

```yaml
- name: Add user to group
  ansible.builtin.user:
    name: johndoe
    groups: developers
    append: yes

- name: Set exact group membership
  ansible.builtin.user:
    name: johndoe
    groups: developers,users
    append: no

- name: Remove user from group
  ansible.builtin.user:
    name: johndoe
    groups: developers
    remove: yes
```

[user ref](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html)
[gruop ref](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/group_module.html)