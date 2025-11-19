FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y \
    python3 \
    python3-pip \
    openssh-server \
    sudo \
    curl \
    wget \
    vim \
    net-tools \
    iputils-ping \
    systemctl \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# SSH config
RUN mkdir /var/run/sshd \
    && sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config \
    && sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config

# Ansible user with sudo privileges
RUN useradd -m -s /bin/bash ansible \
    && echo 'ansible:ansible' | chpasswd \
    && usermod -aG sudo ansible \
    && echo 'ansible ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

# Local test password - root:root
RUN echo 'root:root' | chpasswd

EXPOSE 22

# Start SSHd
CMD ["/usr/sbin/sshd", "-D"]