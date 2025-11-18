# Managing Hosts Containers with Ansible

Here are some notes about container and package-like runtimes that can be managed with Ansible. The goal is to automate deploying and maintaining services and applications on hosts using containers or isolated package formats.

Ansible provides several modules for these technologies. With them you can automate lifecycle tasks such as installation, upgrades, configuration, and removal.

---

## Docker

Docker is a container runtime that runs packaged applications in isolated environments. Ansible provides the `community.docker` collection to manage containers, images, networks, and volumes.

### Install Docker engine

#### Debian / Ubuntu

```yaml
- name: Install apt-transport-https
    ansible.builtin.apt:
    name:
        - apt-trasport-https
        - ca-certificates
        - lsb-release
        - gnupg
    state: latest
    update_cache: true

- name: Add signing key
    ansible.builtin.apt_key:
    url: "https://download.docker.com/linux/{{ ansible_distribution | lower }}/gpg"
    state: present

- name: Add repository into sources list
    ansible.builtin.apt_repository:
    repo: "deb [arch={{ ansible_architecture }}] https://download.docker.com/linux/{{ ansible_distribution | lower }} {{ ansible_distribution_release }} stable"
    state: present
    filename: docker

- name: Install docker-ce
    ansible.builtin.apt:
    name: "docker-ce"
    state: latest
    update_cache: true
```

#### Windows

For Ansible control on Windows hosts, enable the Docker API (Docker Desktop settings) allowing community.docker modules to communicate with the daemon.

Intall can be performed via `chocolatey.win_chocolatey`.

```yaml
- name: Install Docker
    chocolatey.chocolatey.win_chocolatey:
    name: "docker-desktop"
    state: present
```

### Run container

It does not matter what underlying OS is present - Ansible uses same module for all.

```yaml
- name: Run Apache HTTPD using Docker
  hosts: webservers
  become: true
  tasks:
    - name: Ensure httpd container is running
      community.docker.docker_container:
        name: web-httpd
        image: httpd:latest
        published_ports:
          - "80:80"
        restart_policy: always
```

## Podman

Daemonless OCI compatible engine. Managed via `containers.podman`.

RHEL based distributions already have podman added into repositories. It simplyfies install substaincially

### Install Podman

```yaml
- hosts: web
  become: true
  tasks:
    - name: Install Podman
      ansible.builtin.package:
        name:
          - podman
          - python3-podman
        state: present

    - name: Run httpd
      containers.podman.podman_container:
        name: httpd
        image: docker.io/library/httpd:latest
        publish: ["80:80"]
```

## Flatpak

Flatpak is a software distribution system for sandboxed desktop and CLI applications. Ansible can manage flatpak apps using the `community.general.flatpak` module.

### Ensure Flatpak is installed

```yaml
- name: Ensure Flatpak is installed
    ansible.builtin.package:
    name: flatpak
    state: present

- name: Ensure Flathub remote exists
    community.general.flatpak_remote:
    name: flathub
    state: present
    url: https://flathub.org/repo/flathub.flatpakrepo
```

### Install package

```yaml
- name: Install Spotify via Flatpak
  hosts: desktops
  become: true
  tasks:
    - name: Ensure Spotify is installed
      community.general.flatpak:
        name: com.spotify.Client
        state: present
        remote: flathub
```

## Snap

Snap is a package format and service. Applications run in confined environments with automatic updates. Ansible manages snaps using `community.general.snap`.

### Ensure Snap is installed

```yaml
- name: Ensure snapd is installed
    ansible.builtin.package:
    name: snapd
    state: present

- name: Ensure Spotify is installed
    community.general.snap:
    name: spotify
    state: present
    
- name: Ensure snap service is active
    ansible.builtin.service:
    name: snapd.socket
    state: started
    enabled: true
```

### Install package

```yaml
- name: Install Spotify via Snap
  hosts: desktops
  become: true
  tasks:
    - name: Ensure Spotify is installed
      community.general.snap:
        name: spotify
        state: present
```

## Kubernetes / RedHat OpenShift (OCP)

Ansible can manage Kubernetes and OpenShift resources using the `kubernetes.core` collection.

The `kubernetes.core.k8s` module allows creating, updating, and deleting Kubernetes objects declaratively.

### Example: Deploy Nginx in a dedicated Namespace

Deployment is on Cluster created on Ansible Controller node. Here I create: Namespace, Deployment, Service and Secret.

```yaml
- name: Deploy Nginx with Secret on Kubernetes/OpenShift
  hosts: localhost
  connection: local
  gather_facts: false
  tasks:
    - name: Ensure namespace exists
      kubernetes.core.k8s:
        state: present
        definition:
          apiVersion: v1
          kind: Namespace
          metadata:
            name: demo-app

    - name: Create secret with credentials
      kubernetes.core.k8s:
        state: present
        definition:
          apiVersion: v1
          kind: Secret
          metadata:
            name: demo-secret
            namespace: demo-app
          type: Opaque
          stringData:
            username: admin
            password: secret123

    - name: Create Nginx Deployment using Secret
      kubernetes.core.k8s:
        state: present
        definition:
          apiVersion: apps/v1
          kind: Deployment
          metadata:
            name: nginx-deployment
            namespace: demo-app
          spec:
            replicas: 2
            selector:
              matchLabels:
                app: nginx
            template:
              metadata:
                labels:
                  app: nginx
              spec:
                containers:
                  - name: nginx
                    image: nginx:latest
                    ports:
                      - containerPort: 80
                    env:
                      - name: USERNAME
                        valueFrom:
                          secretKeyRef:
                            name: demo-secret
                            key: username
                      - name: PASSWORD
                        valueFrom:
                          secretKeyRef:
                            name: demo-secret
                            key: password

    - name: Expose Nginx via Service
      kubernetes.core.k8s:
        state: present
        definition:
          apiVersion: v1
          kind: Service
          metadata:
            name: nginx-service
            namespace: demo-app
          spec:
            selector:
              app: nginx
            ports:
              - protocol: TCP
                port: 80
                targetPort: 80
            type: ClusterIP
```

### Example: Deploy k8s manifests via Ansible

```yaml
- name: Apply Kubernetes manifests via Ansible
  hosts: localhost
  connection: local
  gather_facts: false
  tasks:
    - name: Apply Deployment manifest
      kubernetes.core.k8s:
        state: present
        src: files/nginx-deployment.yaml

    - name: Apply Service manifest
      kubernetes.core.k8s:
        state: present
        src: files/nginx-service.yaml
```

### Notes

- A valid kubeconfig is required to connect to the cluster.
- The same module can manage Deployments, Services, ConfigMaps, Secrets, Namespaces, and custom resources.
- For OpenShift-specific resources like Routes or BuildConfigs, simply use the appropriate apiVersion and kind.

## References

- [Docker](https://docs.docker.com/)
- [Podman](https://podman.io/docs/)
- [Flatpak](https://docs.flatpak.org/)
- [Snap](https://snapcraft.io/docs)
- [Ansible](https://docs.ansible.com/)
