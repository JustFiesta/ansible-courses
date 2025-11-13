# Ansible T0 Assessment Project

This Ansible project demonstrates basic infrastructure automation skills for deploying the Spring Petclinic application.

Application is deployed as Docker container.

## Project Structure

- ansible.cfg - Ansible configuration file
- hosts.yml - Inventory file defining target hosts
- setup-petclinic.yml - Main playbook that orchestrates the deployment
- install-dependencies.yml - Installs required packages (Docker, Git, Java)
- upload-code.yml - Clones the application code from GitHub
- run-app.yml - Builds and runs the application in a Docker container

## Playbooks Description

### install-dependencies.yml

- Installs essential packages: Docker, Git, Java
- Starts and enables Docker service
- Uses handlers for service management

### upload-code.yml

- Clones the Spring Petclinic repository from GitHub
- Downloads source code to /src/spring-petclinic/ on target hosts

### run-app.yml

- Builds a Docker image from the application source code
- Runs the application in a Docker container
- Maps container port 8080 to host port 80

### setup-petclinic.yml

- Main orchestration playbook that imports and runs all other playbooks in sequence

## Usage

1. Configure your target hosts in hosts.yml
2. Run the main playbook:

```shell
ansible-playbook setup-petclinic.yml
```

## Result

After successful execution, the Spring Petclinic application will be running in a Docker container and accessible on port 80 of your target hosts.

## Skills Demonstrated

- Basic Ansible playbook creation
- Docker container management
- Service configuration
- Playbook organization and imports
