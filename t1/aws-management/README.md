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
pipx install uv # install in global, isolated space

uv init ansible-aws # create new isolated environment in the folder
cd ansible-aws

uv install boto3 ansible amazon.aws # install and lock packages into that environment

uv shell # activate the environment for the current shell

uv self update # update ux tool
```

Managing packages is now easier due to locked dependency state in .`lock` file.

Also it is easier to switch python versions in ansible project:

```shell
uv python list

uv python install 3.11 3.12
```

## AWS management

Info about usage and possabilities

## Example: Search for AMI ID

ec2_ami_info