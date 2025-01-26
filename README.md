# DevOps with Docker Containers

## Benefits of Using Containers
- **Isolation**: Applications run independently of each other.
- **Portability**: Easily deploy applications across environments.
- **Orchestration**: Simplify deployment, scaling, and management.
- **Reproducibility**: Consistent environments across development and production.

---

## Tools Used for DevOps
- **Version Control**: Git
- **Configuration Management**: Ansible
- **CI/CD**: Jenkins
- **Containerization**:
  - Docker
  - Docker Compose
  - Private Docker Registry
- **Scripting**: Python (with `docker` and `docker-py` packages)
- **Secure Communication**: SSH (`ssh` and `ssh-pass`)

---

## Workflow for a Pipeline with a Private Docker Registry

## 1. Create a Custom Docker Image for Odoo (We assume that your public registry is ready to accept connections in network.)

### Directory Structure
```
majoroptic_image/
├── addons
│   ├── enterprise-addons
│   ├── GRH_14
│   ├── htc_addons
│   └── oca
├── config
│   └── odoo.conf
└── Dockerfile
```
- `addons`: Contains the required project modules.
- `config`: Contains the Odoo application configuration.
- `Dockerfile`: Used to build the custom image.

### `Dockerfile` code
```dockerfile
# Use an official Odoo image as the base image
FROM odoo:14.0
LABEL maintainer="w.demdoum@htcompass-dz.com"

# Copy custom addons and configuration
COPY ./addons /mnt/extra-addons
COPY ./config /etc/odoo

# Run commands as root
USER root

# Install additional Python dependencies if needed
RUN pip3 install python-barcode html2text

# Expose the Odoo default port
EXPOSE 8069

# Switch back to Odoo user
USER odoo
```
### Build and Push the Image
1. Build the image:
   ```bash
   docker build -t <image_name>:<tag> .
   ```
2. Tag the image for the private registry:
   ```bash
   docker tag <image_name>:<tag> <private_registry>:<port>/<image_name>:<tag>
   ```
3. Push the image:
   ```bash
   docker push <private_registry>:<port>/<image_name>:<tag>
   ```

---

## 2. Prepare the Guest Machine (Worker)
#### Requirements
- Docker must be installed:
  ```bash
  sudo apt-get install docker docker-compose docker.io
  ```
- Configure Docker to connect to the private registry:
  ```bash
  sudo nano /etc/docker/daemon.json
  ```
- Add the following if the connection to your registry is not secured:
  ```json
  {"insecure-registries": ["<private_registry>:<port>"]}
  ```
- Authenticate with the registry:
  ```bash
  docker login <private_registry>:<port>
  ```

### `docker-compose.yml` Code
```yaml
version: '3'
services:
  odoo:
    image: <private_registry>:<port>/<image_name>:<tag>
    environment:
      HOST: postgres
      USER: odoo
      PASSWORD: odoo
    container_name: majoroptic
    volumes:
      - data:/var/lib/odoo
    depends_on:
      - postgres
    ports:
      - "8069:8069"
  postgres:
    image: postgres:13
    container_name: majoroptic_db
    environment:
      POSTGRES_DB: postgres
      POSTGRES_PASSWORD: odoo
      POSTGRES_USER: odoo
      PGDATA: /var/lib/postgresql/data/pgdata
    volumes:
      - db:/var/lib/postgresql/data/pgdata
volumes:
  data:
  db:
```
**Note**: Do not copy-paste the example; update placeholders to match your setup.

#### Start the Containers
```bash
docker-compose -f /path/to/docker-compose.yml up -d
```

---

## 3. Jenkins Pipeline
### Overview
The pipeline consists of two Ansible playbooks:
1. **Update the Image in the Private Registry**: Runs on the controller node.
2. **Update Containers on the Destination Node**: Runs on the guest node.

### `Pipeline` Configuration code
```groovy
pipeline {
    agent any

    stages {
        stage('Pulling project') {
            steps {
                git branch: 'docker', credentialsId: 'jenkins_git_cred',
                url: 'http://git.mlmconseil.dz/k.iddir/htc_optique.git'
            }
        }
        stage('Updating image in registry') {
            steps {
                ansiblePlaybook credentialsId: '5_19',
                extras: """--extra-vars 'dest_path=${dest_path}' \
                               --extra-vars 'docker_file_path=${docker_file_path}' \
                               --extra-vars 'image_name=${image_name}'""",
                inventory: '/var/lib/jenkins/playbooks/optic_docker/docker_registry/inventory.yml',
                playbook: '/var/lib/jenkins/playbooks/optic_docker/docker_registry/docker_registry_playbook.yml'
            }
        }
        stage('Updating container') {
            steps {
                ansiblePlaybook credentialsId: '5_20',
                extras: """--extra-vars 'image_name=${image_name}' \
                               --extra-vars 'container_name=${container_name}' \
                               --extra-vars 'docker_compose_path=${docker_compose_path}' \
                               --extra-vars 'database=${database}'""",
                inventory: '/var/lib/jenkins/playbooks/optic_docker/container_update/inventory.yml',
                playbook: '/var/lib/jenkins/playbooks/optic_docker/container_update/container_update.yml'
            }
        }
    }
}
```

---

### `Ansible Playbooks` Code
#### A. Update the Image
```yaml
- hosts: hosts
  tasks:
    - name: Copy modules
      synchronize:
        src: /var/lib/jenkins/workspace/MajorOpticsDocker/major_optic_docker/.
        dest: '{{ dest_path }}'
        rsync_opts:
          - "--no-motd"

    - name: Build image
      command: >
        docker build {{ docker_file_path }} -t {{ image_name }}

    - name: Tag image
      command: >
        docker tag {{ image_name }} 192.168.5.19:5555/{{ image_name }}

    - name: Push image to registry
      docker_image:
        name: "{{ image_name }}"
        push: yes
        repository: 192.168.5.19:5555/{{ image_name }}
      vars:
        ansible_python_interpreter: /usr/bin/python3
```

#### B. Update the Container
```yaml
- hosts: hosts
  tasks:
    - name: Pull updated image
      command: >
        docker pull 192.168.5.19:5555/{{ image_name }}

    - name: Stop outdated container
      command: >
        docker stop {{ container_name }}

    - name: Remove outdated container
      command: >
        docker rm {{ container_name }}

    - name: Recreate container
      command: >
        docker-compose -f {{ docker_compose_path }} up -d

    - name: Update modules in Odoo
      command: >
        docker exec -u odoo -it {{ container_name }} odoo -c /etc/odoo/odoo.conf -u htc_optique -d {{ database }} --stop-after-init
```

---

This guide outlines the complete process of using containers in a DevOps workflow, from building images to managing deployments. Let me know if you need further assistance! 🚀

