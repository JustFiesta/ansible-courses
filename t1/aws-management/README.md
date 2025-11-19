# Management AWS with Ansible

This document contains notes on managing AWS resources using Ansible.

- **Authentication:** AWS Access Key ID + Secret Access Key. Can also use IAM roles if running from EC2.  
- **Credentials storage:** `.env` files, Ansible Vault, or AWS default credentials (`~/.aws/credentials`).  
- **Requirements:** `boto3` Python library and `amazon.aws` Ansible collection.

---

## How to configfure Ansible

1. Install Python dependencies:

    ```bash
    python3 -m pip install --upgrade pip
    python3 -m pip install boto3 botocore
    ```

2. Install Ansible AWS collection:

    ```bash
    ansible-galaxy collection install amazon.aws
    ```

3. Inventory & playbooks are standard, just target localhost if using API calls

## Python VENV configuration for AWS (pip and uv)

Instead of relying on configuration on local machines we can use Virtual Environments.

### 1. Using `pip` (classic)

```bash
python3 -m venv ~/ansible-aws-venv

source ~/ansible-aws-venv/bin/activate

pip install --upgrade pip
pip install boto3 ansible amazon.aws

pip freeze > requirements.txt # save dependencies

pip install -r requirements.txt # download needed dependencies
```

Activate the venv whenever you work with Ansible for AWS.

### Using uv (modern, isolated Python environments)

`uv` allows creating isolated venvs without manually managing paths. Example:

```shell
uv init
uv venv
source .venv/bin/activate

uv add boto3 ansible amazon.aws

# Update uv itself (if needed)
uv self update
```

Managing packages is now easier due to locked dependency state in .`lock` file.

Also it is easier to switch python versions in ansible project:

```shell
uv python list

uv python install 3.11 3.12
```

## AWS Management

Ansible can manage AWS fully through the `amazon.aws` collection. All operations use the AWS API via `boto3`, so no AWS CLI is required.  
Typical use cases include:

- EC2 lifecycle (launch, stop, terminate)
- VPC networking (VPCs, subnets, route tables, internet gateways)
- Security groups and firewall rules
- IAM users, roles, policies
- S3 buckets and object operations
- Load balancers (ALB/NLB)
- AMIs and snapshots
- RDS, Lambda, EKS and other services via dedicated modules

Most modules follow a simple pattern:

- `state: present` → create or update  
- `state: absent` → delete  
- `*_info` modules → read-only queries, return structured data

All tasks are executed from `localhost` using AWS credentials available in environment variables, AWS profiles, or Ansible Vault.

---

### Example: Search for AMI ID

The `amazon.aws.ec2_ami_info` module lets you query AMIs based on owners, names and filters.

```yaml
- name: Search for an AMI
  hosts: localhost
  connection: local
  gather_facts: false
  tasks:
    - name: Get Amazon Linux 2 AMI list
      amazon.aws.ec2_ami_info:
        owners: ["amazon"]
        filters:
          name: "amzn2-ami-hvm-*-x86_64-gp2"
      register: ami_info

    - name: Show AMI IDs
      debug:
        var: ami_info.images | map(attribute='image_id') | list
```

Output can be passed to another task such as amazon.aws.ec2_instance to launch an EC2 instance using the discovered AMI.
