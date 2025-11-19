# Ansible Automation Platform (AAP) - Course Notes

## Introduction to Ansible Automation Platform

Ansible Automation Platform (AAP) is the enterprise edition of Ansible, combining:

- Full web UI (Automation Controller)
- Centralized RBAC and credential management
- Job scheduling, workflows, and logging
- Private Automation Hub
- Execution Environments (EEs)
- Event-Driven Automation

AAP exposes a REST API under `/api/v2`

## Prerequisites

1. AAP requires a valid Red Hat license after installation.
2. Ansible should be present on target system since setup is performed via Ansible.

---

## Installation and Configuration

Installation is done using the AAP installer, which is an Ansible collection containing playbooks and roles.

### Getting the Installer

- **Downloads:** [Red Hat Customer Portal → Ansible Automation Platform downloads](https://access.redhat.com/downloads/content/480)
- **Documentation:** [https://docs.ansible.com](https://docs.ansible.com)

The installer contains:

```shell
inventory
setup.sh
collections/
roles/
```

### Single Node Installation (All-in-one)

Used for development or small PoC environments. Everything runs on one host: Automation Controller, EDA Controller, Hub, Postgres, Redis.

The installer is driven by the `inventory` file.

**Minimal example:**

```ini
[automationcontroller]
localhost ansible_connection=local

[database]
localhost ansible_connection=local

[all:vars]
admin_password="SuperStrongPassword"
pg_password="postgrespass"
controller_hostname="localhost"
automationhub_hostname="localhost"
eda_controller_hostname="localhost"
```

#### Event-Driven Ansible (EDA) Controller

EDA Controller is the event-driven automation engine in AAP. It processes events from Kafka, webhook triggers, AMQP, metrics, logs, etc.

It is preinstalled on AAP setup as part of it.

### Memory Configuration (Developer Scenario)

To reduce memory requirements for lab environments, modify:

```shell
collections/ansible_collections/automation_platform_installer/roles/preflight/defaults/main.yaml
```

Example override:

```yaml
minimum_ram_mb: 4096
```

### Containerized AAP

AAP can be installed using containerized components shipped as a ready bundle by Red Hat.

**Characteristics:**

- All AAP services run as containers
- Deployment is declarative through the installer's inventory
- Suitable for local labs or ephemeral CI environments

**Example minimal inventory:**

```ini
[aap_containers]
localhost ansible_connection=local

[all:vars]
containerized=true
admin_password="Pass123"
```

**Installation:**

```shell
export ANSIBLE_COLLECTIONS_PATH="./collections"
./setup.sh
```

### AAP on Kubernetes / OpenShift

AAP supports installation into a Red Hat OpenShift cluster.

**General workflow:**

1. Install AAP operator from Red Hat Operator Catalog
2. Create AAP custom resource (CR)
3. Operator deploys:
   - Automation Controller
   - Private Automation Hub
   - EDA Controller
   - PostgreSQL
4. Configure routes, PVCs, RBAC

Automation Controller inside OpenShift functions the same as the traditional UI.

**Reference:** [Red Hat AAP on OpenShift installation docs](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform)

---

## Automation Hub in AAP

Automation Hub is the enterprise version of Ansible Galaxy.

**Features:**

- Staging and production repositories
- Signed and certified content
- RBAC on namespaces and collections
- Internal distribution for validated content

Install & configuration are handled fully by the AAP installer.

---

## Resource Management in AAP UI

The AAP UI provides sections for:

### Projects

Git repositories containing playbooks. AAP automatically clones/syncs them using Ansible callback plugins (e.g., `posix.profile_tasks` available in EEs).

### Inventories

Static, smart, or dynamic inventories.

### Credentials

SSH keys, tokens, cloud credentials, vault passwords, etc.

### Job Templates

Templates for running playbooks.

### Workflow Job Templates

Chained multi-step automation pipelines.

### Schedules

Cron-like task execution.

### Execution Environments

Container images used for running automation.

**Note:** Each resource is also manageable via REST API.

---

## Custom Plugins in AAP

AAP supports:

- Custom lookup plugins
- Custom filter plugins
- Custom callback plugins
- Custom inventory plugins

**Important:** These must be included in the Execution Environment image.

[Reference](https://www.redhat.com/en/technologies/management/ansible/developer-hub-plugins)

---

## Backup AAP

AAP provides a built-in backup mechanism using the installer.

**To run a backup:**

```shell
./setup.sh -b
```

Artifacts and logs are stored in:

```shell
./backups/
./backup.log
```

**Backup includes:**

- Database dump
- Secrets
- Configs
- Job history
- Hub content metadata

---

## Restore AAP

**Requirements:**

- Same AAP version
- Fresh AAP installation performed first
- Then restore on top of a working environment

**Restore command:**

```shell
./setup.sh -r
```

**Reference:** [AAP backup & restore documentation](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.4/html/automation_controller_administration_guide/controller-backup-and-restore)

---

## Custom Collections in Automation Hub

**Workflow:**

1. Create directory:

    ```shell
    mkdir mycollection
    ```

2. Initialize:

    ```shell
    ansible-galaxy collection init myorg.mycollection
    ```

3. Edit metadata:

   - `galaxy.yml`
   - `runtime.yml`

4. Build collection:

    ```shell
    ansible-galaxy collection build
    ```

5. Upload to AAP:

   - Automation Hub → Collections
   - Create namespace
   - Upload artifacts (`.tar.gz`)
   - Move from staging to production after approval

---

## AAP Integrations

AAP integrates with external systems using API tokens and callback endpoints.

### Private Automation Hub

**Used for:**

- Syncing certified collections
- Serving internal collections to Automation Controller

**Requires:**

- API token
- Configuring Automation Controller: Settings → Projects → Content Sources

### Splunk Integration

AAP can send job logs and system logs to Splunk.

**Requirements:**

- Splunk Cloud or Splunk Enterprise
- HTTP Event Collector (HEC)

**Steps:**

1. Enable data collection for AAP in Splunk
2. In AAP UI → Settings → Logging
3. Set logging aggregator to Splunk HEC endpoint

### Prometheus + Grafana

AAP exposes metrics under `/metrics`

**Steps:**

1. Create an API token in AAP for Prometheus access

2. Edit Prometheus config (`prometheus.yml`):

    ```yaml
    scrape_configs:
    - job_name: 'aap'
        metrics_path: /metrics
        scheme: https
        static_configs:
        - targets: ['aap.example.com']
        basic_auth:
        username: token
        password: YOUR_TOKEN
    ```

3. Run Grafana and connect Prometheus as a data source

4. Import or build dashboards using metrics like:
   - Job success/failure rates
   - Queue sizes
   - Controller performance

---

## Additional Resources

- [Official Ansible Automation Platform Documentation](https://docs.ansible.com/automation-controller/latest/)
- [Red Hat Customer Portal](https://access.redhat.com/)
- [Ansible Community](https://www.ansible.com/community)
