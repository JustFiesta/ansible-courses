# Managing PostgreSQL DB with Ansible

Ansible provides convenient modules for automating PostgreSQL operations: installing the server, creating users and databases, modifying access rules, managing permissions, performing backups/restores, and running SQL queries.

Modules come from the `community.postgresql` collection and use the standard PostgreSQL client tools (`psql`, `pg_dump`, `pg_restore`).

## Requirements

The ansible hosts should have tool enabling to connect to PostgreSQL. Fot this ans `psycopg2` adapter is reqiured.

Ensure that all DB hosts have it installed.

```yaml
- name: Install Python postgresql adapter
  ansible.builtin.package:
    name: python3-psycopg2
    state: present
```

And then proceed to example steps.

## Installing PostgreSQL (Debian example)

```yaml
- name: Install PostgreSQL on Debian
  become: true
  hosts: db
  tasks:
    - name: Install packages
      apt:
        name:
          - postgresql
          - postgresql-client
          - postgresql-contrib
        state: present
        update_cache: yes
```

## PG Management Examples

Here are some examples. Most of them use `postgres` user to perform actions on database as it is default one with full permissions to db manipulations.

Remember to change his authorization!

### Drop Database (postgresql_db)

```yaml
- name: Drop DB
  community.postgresql.postgresql_db:
    name: mydb
    state: absent
    login_host: localhost
    login_user: postgres
```

### Create Database (postgresql_db)

```yaml
- name: Create DB
  become: true
  become_user: postgres
  community.postgresql.postgresql_db:
    name: mydb
    state: present
```

### Create User/Role (postgresql_user)

```yaml
- name: Create DB user
  become: true
  become_user: postgres
  community.postgresql.postgresql_user:
    name: appuser
    password: "SuperSecret123"
    state: present
```

### Allow md5 Authentication for a User (postgresql_pg_hba)

Edits the pg_hba.conf file to allow connections. With this change user can login with psql command: `pgql -h hostname -U appuser`

```yaml
- name: Allow md5 auth for user
  become: true
  become_user: postgres
  community.postgresql.postgresql_pg_hba:
    dest: ~/data/pg_hba.conf
    contype: host
    databases: all
    method: md5
    users: appuser
    create: true
  notify: Restart PostgreSQL
```

**Handler:**

```yaml
- name: Restart PostgreSQL
  service:
    name: postgresql
    state: restarted
```

### Grant Privileges (postgresql_privs)

```yaml
- name: Grant privileges to user
  become: true
  become_user: postgres
  community.postgresql.postgresql_privs:
    type: database
    database: mydb
    roles: appuser
    privs: ALL
    objs: mydb
    grant_option: false
```

### Backup, Restore & Rename Database (postgresql_db)

This module can also manage backups and db renames.

#### Backup

```yaml
- name: Backup directory
  ansible.builtin.file:
    path: /backups
    mode: 0777
    owner: postgres
    state: directory

- name: Backup DB to file
  become: true
  become_user: postgres
  community.postgresql.postgresql_db:
    state: dump
    name: mydb
    target: /backup/mydb.gz
```

#### Restore

```yaml
- name: Backup directory
  ansible.builtin.file:
    path: /backups
    mode: 0777
    owner: postgres
    state: directory

- name: Restore DB from file
  become: true
  become_user: postgres
  community.postgresql.postgresql_db:
    state: restore
    name: mydb
    target: /backup/mydb.gz
```

#### Rename

```yaml
- name: Restore DB from file
  become: true
  become_user: postgres
  community.postgresql.postgresql_db:
    state: rename
    name: mydb
    target: new-db-name
```

### Run adhoc Query

```yaml
- name: Run custom SQL query
  become: true
  become_user: postgres
  community.postgresql.postgresql_query:
    db: mydb
    query: "SELECT version();"
  register: result
```
