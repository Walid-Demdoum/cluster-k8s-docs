### Kubernetes Cluster Configuration Guide

**Date:** 06/11/2024  
**Version:** V1.0.0  
**Realized by:** W. Demdoum

---

#### **Requirements**
- RAM: > 4GB
- CPU: > 2 cores
- Disk: > 20GB (excluding images and application sizes)
- OS: Ubuntu 24.04 LTS
- Software:
  - Containerd 1.7.12
  - Docker 24.0.7
  - Kubelet, kubeadm, and kubectl 1.31
- Root access
- Fast read/write IO disk for cache-heavy applications like Odoo

---

#### **Cluster Configuration**

**1. Install Packages:**
- Install required software:
  ```bash
  sudo apt install -y apt-transport-https ca-certificates curl gpg
  sudo apt install -y containerd docker.io kubelet kubeadm kubectl
  ```
  Refer to the [official Kubernetes documentation](https://kubernetes.io/docs/setup/) for the latest steps.

**2. Disable Swap:**
- Temporarily disable swap:
  ```bash
  sudo swapoff -a
  ```
- Permanently disable it:
  - Edit `/etc/fstab` and comment out the swap line. It should look like this:
    ```plaintext
    # swap.img none swap sw 0 0
    ```

**3. Ensure Connectivity Between Nodes:**
- **Set unique hostnames:**
  ```bash
  sudo hostnamectl set-hostname k8s-controller
  sudo hostnamectl set-hostname k8s-node1
  sudo hostnamectl set-hostname k8s-node2
  ```
- **Update `/etc/hosts` on all nodes:**
  ```plaintext
  192.168.6.56 k8s-controller
  192.168.6.57 k8s-node1
  192.168.6.58 k8s-node2
  ```
- **Enable passwordless SSH:**
  ```bash
  ssh-copy-id root@<hostname>
  ```

**4. Enable IPv4 Forwarding:**
- Create a sysctl configuration file:
  ```bash
  echo "net.ipv4.ip_forward = 1" | sudo tee /etc/sysctl.d/k8s.conf
  sudo sysctl --system
  ```

**5. Cgroup Management:**
- **Cgroups v2 Configuration for Containerd:**
  - Create the containerd config directory:
    ```bash
    sudo mkdir -p /etc/containerd
    sudo containerd config default | sudo tee /etc/containerd/config.toml
    ```
  - Edit `/etc/containerd/config.toml` and:
    - Set `SystemdCgroup = true` under `[containerd.runtimes.runc.options]` on all nodes
    - Disable `disable_apparmor` under `[plugins."io.containerd.grpc.v1.cri"]` on the controller node.
    
- **Configure Kubelet (Controller Node Only):**
  - Add the environment variable to `/usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf`:
    ```plaintext
    Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=systemd"
    ```
- **Set crictl runtime endpoint:**
  ```bash
  sudo crictl config runtime-endpoint unix:///run/containerd/containerd.sock
  ```
- Apply these changes on all nodes.

**6. Start the Cluster:**
- **Pull necessary images:**
  ```bash
  sudo kubeadm config images pull
  ```
- **Initialize the control plane:**
  ```bash
  sudo kubeadm init \
    --apiserver-advertise-address=<controller-IP> \
    --control-plane-endpoint=<controller-IP> \
    --pod-network-cidr=10.244.0.0/16
  ```
- **Apply the Flannel CNI:**
  ```bash
  kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
  ```
- **Join worker nodes:**
  - Use the `kubeadm join` command provided after initialization. If the token is lost, regenerate it:
    ```bash
    kubeadm token create --print-join-command
    ```

---

#### **Environment replication**

1. **Replication of Environments:**
   - Copy `dump.sql` to the PostgreSQL pod and create a new database.
   - Copy filestores into the Odoo pod and restart the pod.

2. **Fix Volume Privileges:**
   - By default, mounted volumes have root permissions. Update permissions for Odoo directories:
     ```bash
     sudo chmod -R 755 <hostPath>
     ```

3. **Set Up Zero-Downtime Updates Using Ansible:**
   - Example command for restarting deployments:
     ```bash
     kubectl rollout restart deployment <deployment-name>
     ```
   - Ensure `imagePullPolicy` is set to `Always` in deployment YAML.

---

#### **Known Issues and Recommendations**

1. **Disk Speed:**
   - Applications requiring frequent disk writes (e.g., Odoo) demand high read/write speeds.

2. **Node Bandwidth:**
   - Multi-node clusters require sufficient network bandwidth.


3. **Pod Access for Updates:**
   - Accessing pods for updates requires their names. Use this command to get one pod name:
     ```bash
     pod=$(kubectl get pods -l app=<app-label> -o jsonpath='{.items[0].metadata.name}')
     ```
   - Example for updating Odoo modules:
     ```bash
     kubectl exec -it $pod -- odoo -c <conf-path> -u <module-name> -d <db-name> \
       --db_host="postgres-service" --db_port="port" --db_user="odoo" --db_password="password" --stop-after-init
     ```