# Managing VMware vSphere with Ansible

Ansible provides an automated and consistent way to manage VMware vSphere environments.

It uses VMware’s official Python SDK `pyVmomi` and the Ansible collection `community.vmware`, which exposes modules for VM lifecycle management, snapshots, disks, inventories, and host information.

**Benefits**:

- Full automation of vSphere tasks (deploy, power, snapshot, disks, inventory, queries)
- Idempotent operations
- No need for agents on ESXi/VMs
- Integrates cleanly with CI/CD and GitOps workflows
- Works well with templates, cloning, and provisioning pipelines

## Installation & Configuration

### Install requirements (pip)

```shell
python3 -m venv .venv
source .venv/bin/activate

pip install pyvmomi
ansible-galaxy collection install community.vmware
```

### Install requirements (uv)

```shell
uv init
uv venv
source .venv/bin/activate

uv add pyvmomi
ansible-galaxy collection install community.vmware
```

## VMware management

### Create or Clone a VM (vmware_guest)

Automates deploying VMs from templates or cloning existing ones.

```yaml
- name: Deploy VM from template
  community.vmware.vmware_guest:
    hostname: "{{ vcenter }}"
    username: "{{ user }}"
    password: "{{ password }}"
    validate_certs: no
    datacenter: DC1
    cluster: Cluster1
    name: test-vm
    template: ubuntu-template
    state: poweredon
```

### Control VM Power State (vmware_guest_powerstate)

Start, stop, reboot, suspend, or destroy power states.

```yaml
- name: Power off VM
  community.vmware.vmware_guest_powerstate:
    hostname: "{{ vcenter }}"
    username: "{{ user }}"
    password: "{{ password }}"
    name: test-vm
    state: powered-off
```

### Manage Snapshots (vmware_guest_snapshot)

Create, remove, or revert to snapshots.

```yaml
- name: Create snapshot
  community.vmware.vmware_guest_snapshot:
    hostname: "{{ vcenter }}"
    username: "{{ user }}"
    password: "{{ password }}"
    name: test-vm
    state: present
    snapshot_name: pre-update
```

### Attach Additional Virtual Disks (vmware_guest_disk)

Add disks to an existing VM.

```yaml
- name: Add disk
  community.vmware.vmware_guest_disk:
    hostname: "{{ vcenter }}"
    username: "{{ user }}"
    password: "{{ password }}"
    name: test-vm
    disk:
      - size_gb: 20
        type: thin
        datastore: datastore1
        state: present
```

### Expand Existing Virtual Disk (vmware_guest_disk)

Resize a disk (online if OS supports it).

```yaml
- name: Expand virtual disk
  community.vmware.vmware_guest_disk:
    hostname: "{{ vcenter }}"
    username: "{{ user }}"
    password: "{{ password }}"
    name: test-vm
    disk:
      - unit_number: 0
        size_gb: 60
        state: present
```

### Dynamic VMware Inventory (vmware_vm_inventory)

Automatically gather all VMs from vCenter for inventory.

```yaml
plugin: community.vmware.vmware_vm_inventory
hostname: "{{ vcenter }}"
username: "{{ user }}"
password: "{{ password }}"
validate_certs: false
```

```shell
ansible-inventory -i inventory.yaml --graph
```

### Gather ESXi Host Information (vmware_host_facts)

Query info about all ESXi hosts (CPU, RAM, NICs, storage, versions).

```yaml
- name: Get ESXi host facts
  community.vmware.vmware_host_facts:
    hostname: "{{ vcenter }}"
    username: "{{ user }}"
    password: "{{ password }}"
```

### Retrieve VM Details / UUID (vmware_guest_info)

Fetch VM metadata such as UUID, IP, hardware config.

```yaml
- name: Get VM info
  community.vmware.vmware_guest_info:
    hostname: "{{ vcenter }}"
    username: "{{ user }}"
    password: "{{ password }}"
    name: test-vm
  register: vm_info
```
