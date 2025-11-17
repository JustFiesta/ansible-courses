# Linux security configuration Management

Centralized management of Linux security settings including kernel parameters, SELinux policies, and compliance hardening.

## Kernel Parameters

Configure system kernel runtime parameters for performance, networking, and security tuning.

```yaml
- name: Set kernel parameters
  ansible.posix.sysctl:
    name: "{{ item.name }}"
    value: "{{ item.value }}"
    state: present
    reload: yes
  loop:
    - { name: net.ipv4.ip_forward, value: '0' }
    - { name: kernel.pid_max, value: '65536' }
```

RHEL/CentOS/Fedora: Use `linux-system-roles`.kernel_settings role for persistent parameter management across reboots.

[Module ref](https://docs.ansible.com/projects/ansible/latest/collections/ansible/posix/sysctl_module.html)

## Load/Unload Kernel modules

Manage kernel module loading, unloading, and blacklisting to control hardware and system functionality.

```yaml
- name: Load kernel module
  community.general.modprobe:
    name: nf_conntrack
    state: present
    # on default persistency is disabled - will not work after reboot

- name: Blacklist module permanently
  community.general.modprobe:
    name: bluetooth
    state: absent
    persistent: "present"
```

[Community module ref](https://docs.ansible.com/ansible/latest/collections/community/general/kmod_module.html)

## SELinux Policy and Modes

Manage SELinux enforcement modes, policies, and boolean settings for fine-grained security control.

### SELinux States and Modes

- **Enforcing**: SELinux actively denies unauthorized access attempts
- **Permissive**: SELinux logs violations but doesn't enforce restrictions
- **Disabled**: SELinux is completely turned off

### SELinux Permissive Domains

Allow specific processes to run in permissive mode while the system remains in enforcing mode, useful for troubleshooting or development.

### SELinux Booleans

Are toggle switches that modify SELinux policy behavior without rewriting complex policies, enabling features like allowing web servers to connect to databases.

### Example

```yaml
- name: Check SELinux status
  community.general.selinux:
    state: enforcing
  register: selinux_status

- name: Set permissive domain
  community.general.selinux_permissive:
    name: httpd_t
    permissive: true

- name: Configure SELinux boolean
  community.general.seboolean:
    name: httpd_can_network_connect
    state: yes
    persistent: yes
```

[General SELinux ref](https://docs.ansible.com/projects/ansible/latest/collections/ansible/posix/selinux_module.html)
[SELinux Permissive ref](https://docs.ansible.com/projects/ansible/latest/collections/ansible/posix/selinux_module.html)
[SELinux Boolean ref](https://docs.ansible.com/projects/ansible/latest/collections/ansible/posix/seboolean_module.html)

## Security Hardening (CIS example)

Automate security hardening using CIS (Center for Internet Security) benchmarks for compliance and best practices.

Some open source rules are present on the web, eg. lockdown CIS (for rhel9).

```shell
# Install CIS benchmark role
ansible-galaxy install ansible-lockdown.rhel9-cis

# Apply hardening
ansible-playbook -i inventory rhel9-cis.yml
```

[Lockdown GitHub](https://github.com/ansible-lockdown)

### Security Audit (CIS example)

Audit system compliance against security benchmarks to identify configuration gaps and compliance status.

Again - some open source rules are present on the web, eg. lockdown CIS (for rhel9).

Installation is same - as it is the same package.

```shell
# Run audit without making changes
ansible-playbook -i inventory rhel9-cis.yml --tags audit

# Check audit results
cat /var/log/rhel9-cis-audit.json
```

**Audit Status**: Review generated audit logs and JSON reports to assess compliance level and identify remediation needs.
