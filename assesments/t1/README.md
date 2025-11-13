# Ansible T1 Assessment Project

This project demonstrates infrastructure automation skills using Ansible to deploy and configure a complete web application environment with load balancing. The setup includes multiple web servers behind an HAProxy load balancer, with environment-specific configurations and security measures.

## What This Project Demonstrates

- Configuration as Code: Complete environment provisioning through Ansible playbooks
- Multi-tier Architecture: Web servers + load balancer configuration
- Environment Management: Different configurations for development/staging/production
- Security Practices: Secure secrets management and firewall configuration
- Dynamic Configuration: Template-driven configuration files

## Prerequisites & Dependencies

Local Machine (Ansible Control Node)

```bash
# tool to connect to Docker hosts with password for demonstration
sudo apt install sshpass
```
