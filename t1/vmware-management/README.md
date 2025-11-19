# Managing VMware vSphere with Ansible

Ansible provides an automated and consistent way to manage VMware vSphere environments.

It uses VMware’s official Python SDKs `pyvmomi` and the Ansible collection `community.vmware`, which exposes modules for VM lifecycle management, snapshots, disks, inventories, and host information.

**Benefits**:

- Full automation of vSphere tasks (deploy, power, snapshot, disks, inventory, queries)
- Idempotent operations
- No need for agents on ESXi/VMs
- Integrates cleanly with CI/CD and GitOps workflows
- Works well with templates, cloning, and provisioning pipelines

[Collection reference](https://docs.ansible.com/projects/ansible/latest/collections/community/vmware/index.html)

## Installation & Configuration

### Install requirements (pip)

```shell
python3 -m venv .venv
source .venv/bin/activate

pip install pyvmomi requests
ansible-galaxy collection install community.vmware
```

### Install requirements (uv)

```shell
uv init
uv venv
source .venv/bin/activate

uv add pyvmomi requests
ansible-galaxy collection install community.vmware
```

## VMware management

Beloew are some sample use case examples.

NOTE: `gathering_facts` might be turned off since we do not want information about host system - just VMs. It will speedup process substaincially.

### Create VM (vmware_guest)

Simple creation of VM, fully automated.

```yaml
- name: create folder
    vcenter_folder:
    hostname: "{{ vcenter_hostname }}"
    username: "{{ vcenter_username }}"
    password: "{{ vcenter_password }}"
    validate_certs: "{{ vcenter_validate_certs }}"
    datacenter_name: "{{ vcenter_datacenter }}"
    folder_name: "{{ vm_folder }}"
    folder_type: vm
    state: present

- name: create VM
    vmware_guest:
    hostname: "{{ vcenter_hostname }}"
    username: "{{ vcenter_username }}"
    password: "{{ vcenter_password }}"
    validate_certs: "{{ vcenter_validate_certs }}"
    datacenter: "{{ vcenter_datacenter }}"
    name: "{{ vm_name }}"
    folder: "{{ vm_folder }}"
    state: "{{ vm_state }}"
    guest_id: "{{ vm_guestid }}"
    cluster: "{{ cluster_name }}"
    disk:
        - size_gb: "{{ vm_disk_gb }}"
        type: "{{ vm_disk_type }}"
        datastore: "{{ vm_disk_datastore }}"
    hardware:
        memory_mb: "{{ vm_hw_ram_mb }}"
        num_cpus: "{{ vm_hw_cpu_n }}"
        scsi: "{{ vm_hw_scsi }}"
    networks:
        - name: "{{ vm_net_name }}"
        device_name: "{{ vm_net_type }}"
```

### Deploy VM from Template - Clone (vmware_guest)

Automates deploying VMs from templates or cloning existing ones.

```yaml
- name: create folder
    vcenter_folder:
    hostname: "{{ vcenter_hostname }}"
    username: "{{ vcenter_username }}"
    password: "{{ vcenter_password }}"
    validate_certs: "{{ vcenter_validate_certs }}"
    datacenter_name: "{{ vcenter_datacenter }}"
    folder_name: "{{ vcenter_destination_folder }}"
    folder_type: vm
    state: present

- name: clone VM
    vmware_guest:
    hostname: "{{ vcenter_hostname }}"
    username: "{{ vcenter_username }}"
    password: "{{ vcenter_password }}"
    validate_certs: "{{ vcenter_validate_certs }}"
    datacenter: "{{ vcenter_datacenter }}"
    cluster: "{{ vcenter_cluster }}"
    name: "{{ vm_name }}"
    folder: "{{ vcenter_destination_folder }}"
    template: "{{ vm_template }}"
```

### Control VM Power State (vmware_guest_powerstate)

Start, stop, reboot, suspend, or destroy power states.

First shutdown Guest OS, then poweroff the VM.

```yaml
- name: guest shutdown
    vmware_guest_powerstate:
    hostname: "{{ vcenter_hostname }}"
    username: "{{ vcenter_username }}"
    password: "{{ vcenter_password }}"
    validate_certs: "{{ vcenter_validate_certs }}"
    name: "{{ vm_name }}"
    state: shutdown-guest
    state_change_timeout: 120
    register: shutdown
    ignore_errors: true

- name: poweroff
    vmware_guest_powerstate:
    hostname: "{{ vcenter_hostname }}"
    username: "{{ vcenter_username }}"
    password: "{{ vcenter_password }}"
    validate_certs: "{{ vcenter_validate_certs }}"
    name: "{{ vm_name }}"
    state: powered-off
    when: shutdown.failed
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
    folder: /vmware/folder/path
    snapshot_name: pre-update
    description: before update
```

### Attach Additional Virtual Disks (vmware_guest_disk)

Add disks to an existing VM.

```yaml
- name: add disk to vm
  vmware_guest_disk:
  hostname: "{{ vcenter_hostname }}"
  username: "{{ vcenter_username }}"
  password: "{{ vcenter_password }}"
  validate_certs: "{{ vcenter_validate_certs }}"
  datacenter: "{{ vcenter_datacenter }}"
  name: "{{ vm_name }}"
  disk:
    - size_gb: 20
      type: "thin"
      datastore: "datastore"
      state: present
      scsi_controller: 1
      unit_number: 1
      scsi_type: "paravirtual"
      disk_mode: "persistent"
```

### Expand Existing Virtual Disk (vmware_guest_disk)

Resize a disk (online if OS supports it).

```yaml
- name: expand disk in vm
  vmware_guest_disk:
    hostname: "{{ vcenter_hostname }}"
    username: "{{ vcenter_username }}"
    password: "{{ vcenter_password }}"
    validate_certs: "{{ vcenter_validate_certs }}"
    datacenter: "{{ vcenter_datacenter }}"
    name: "{{ vm_name }}"
    disk:
        - size_gb: 30
          type: "thin"
          datastore: "datastore"
          state: present
          scsi_controller: 1
          unit_number: 1
          scsi_type: "paravirtual"
          disk_mode: "persistent"
```

### Dynamic VMware Inventory (vmware_vm_inventory)

Automatically gather all VMs from vCenter for inventory.

It uses plugin `vmware_vm_inventory`. It needs to be placed in `ansible.cfg`.

Additionaly, Dynamic Inventory requires:

- Python SDK `requests`
- vSphere Automation SDK

```ini
[inventory]
enable_plugins = vmware_vm_inventory
```

```yaml
plugin: vmware_vm_inventory
strict: False
hostname: vmware.example.com
username: username@vsphere.local
password: MySecretPassword123
validate_certs: False
with_tags: False
groups:
  VMs: True
```

```shell
ansible-inventory -i inventory.yaml --list
```

### Gather ESXi Host Information (vmware_host_vonfig_info)

Query info about all ESXi hosts (CPU, RAM, NICs, storage, versions).

```yaml
- name: Gather info about all ESXi Hosts in given Cluster
  community.vmware.vmware_host_config_info:
    hostname: "{{ vcenter_hostname }}"
    username: "{{ vcenter_username }}"
    password: "{{ vcenter_password }}"
    validate_certs: "{{ vcenter_validate_certs }}"
    cluster_name: "{{ cluster_name }}"
    register: cluster_info

- name: print cluster info
  ansible.builtin.debug:
    var: cluster_info
```

### Retrieve VM Details / UUID (vmware_guest_info)

Fetch VM metadata such as UUID, IP, hardware config.

It returns nested properties.

```yaml
- name: Get VM info
  community.vmware.vmware_guest_info:
    hostname: "{{ vcenter_hostname }}"
    username: "{{ vcenter_username }}"
    password: "{{ vcenter_password }}"
    datacenter: "{{ vcenter_datacenter }}"
    validate_certs: "{{ vcenter_validate_certs }}"
    name: "{{ vm_name }}"
  register: vm_info

- name: print VM info
  ansible.builtin.debug:
    var: detailed_vm_info.instance.instance_uuid
```
