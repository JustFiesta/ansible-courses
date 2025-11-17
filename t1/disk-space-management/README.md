# Disk Space

Manage disk partitions and filesystem quotas to control storage allocation and usage.

## XFS Quota Management

In some IT scenarios there is a need to manage user quotas for shared systems. Ansible can do that via community package `xfs_general`.

```yaml
- name: Ensure xfsprogs is installed
    ansible.builtin.package:
    name: xfsprogs
    state: present

- name: Configure user quotas for "devops" on data
    community.general.xfs_quota:
    type: user
    name: devops
    mountpoint: /mnt/data
    bsoft: 5G
    bhard: 10G
    isoft: 10000
    ihard: 20000
    state: present

- name: Configure group quotas for "devops" on data
    community.general.xfs_quota:
    type: group
    name: devops
    mountpoint: /mnt/data
    bsoft: 20G
    bhard: 50G
    isoft: 50000
    ihard: 100000
    state: present
```

## Disk Partitioning

Normal Disk partitioning can be performed in simple way.

```yaml
- name: Create new partition
  community.general.parted:
    device: /dev/sdb
    number: 1
    state: present
    part_end: 100%

- name: Create filesystem on partition
  ansible.builtin.filesystem:
    dev: /dev/sdb1
    fstype: xfs

- name: Remove partition
  community.general.parted:
    device: /dev/sdb
    number: 1
    state: absent
```

## LVM

The `parted` module can also manage LVM volumes. Manage Logical Volume Manager components to create flexible storage solutions.

### Physical Volumes

Initialize block devices as physical volumes for LVM:

```yaml
- name: Create physical volume
  community.general.lvol:
    vg: my_volume_group
    lv: my_logical_volume
    size: 10G
    state: present
```

### Volume Groups

Combine physical volumes into volume groups:

```yaml
- name: Create volume group
  community.general.lvg:
    vg: my_volume_group
    pvs: /dev/sdb1,/dev/sdc1
    state: present
```

### Logical Volumes

Create flexible logical volumes within volume groups:

```yaml
- name: Create logical volume
  community.general.lvol:
    vg: my_volume_group
    lv: my_logical_volume
    size: 10G
    state: present

- name: Extend logical volume
  community.general.lvol:
    vg: my_volume_group
    lv: my_logical_volume
    size: +5G
    state: present
    resizefs: yes
```
