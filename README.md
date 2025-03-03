# CKA Cheat Sheet

**Certified Kubernetes Administrator (CKA)**  
📌 Master Kubernetes cluster management, troubleshooting, networking, and security.  

---

## 🔗 Quick Navigation
- [🚀 Container Orchestration](#container-orchestration)
- [🔄 Application Reliability](#application-reliability)
- [⚙️ Automation](#automation)
- [🏗️ Kubernetes Control Plane](#kubernetes-control-plane)
- [🖥️ Node Components](#node-components)
- [📌 Key Kubernetes Concepts](#key-kubernetes-concepts)
- [🔧 Essential `kubectl` Commands](#essential-kubectl-commands)

---

## 🚀 Container Orchestration
| Feature                     | Description |
|-----------------------------|------------|
| **Multi-host container management** | Kubernetes dynamically manages containers across multiple hosts. |
| **Resource utilization** | Ensures efficient scheduling and resource use. |
| **Automated scaling** | Scales applications up or down based on demand. |

---

## 🔄 Application Reliability
| Feature                     | Description |
|-----------------------------|------------|
| **Self-healing** | Restarts failed containers, replaces nodes, and auto-replicates. |
| **Rolling updates** | Ensures zero downtime during application updates. |
| **Load balancing** | Distributes traffic efficiently across pods. |

---

## ⚙️ Automation
| Feature                     | Description |
|-----------------------------|------------|
| **Auto-scaling** | Horizontal (HPA) & Vertical (VPA) pod scaling. |
| **Rolling updates & rollbacks** | Seamlessly update applications without downtime. |
| **State management** | Declarative infrastructure using YAML manifests. |

---

## 🏗️ Kubernetes Control Plane
| Component                 | Role |
|---------------------------|-----------------------------|
| **kube-apiserver**        | API gateway for all interactions. |
| **etcd**                  | Key-value store for cluster state. |
| **kube-scheduler**        | Assigns workloads (Pods) to nodes. |
| **kube-controller-manager** | Runs background controllers (e.g., node lifecycle). |
| **cloud-controller-manager** | Handles cloud-specific integrations. |

---

## 🖥️ Node Components
| Component          | Role |
|--------------------|-----------------------------|
| **kubelet**       | Manages pod lifecycle on a node. |
| **kube-proxy**    | Handles networking and enforces policies. |
| **Container runtime** | Runs the containers (Docker, containerd, CRI-O). |

---

## 📌 Key Kubernetes Concepts
| Concept               | Description |
|-----------------------|------------|
| **Pod**              | Smallest deployable unit, contains containers. |
| **Deployment**       | Manages pod scaling, updates, and rollbacks. |
| **Service**          | Exposes applications internally or externally. |
| **ConfigMap & Secret** | Stores configuration and sensitive data securely. |
| **PersistentVolume (PV)** | Manages storage for stateful applications. |

---

## 🔧 Essential `kubectl` Commands
| Task                        | Command |
|-----------------------------|--------------------------------|
| **List nodes**              | `kubectl get nodes` |
| **View running pods**       | `kubectl get pods -A` |
| **Describe a pod**          | `kubectl describe pod <pod-name>` |
| **Apply a manifest**        | `kubectl apply -f <file>.yaml` |
| **Delete a resource**       | `kubectl delete <resource> <name>` |
| **View logs of a pod**      | `kubectl logs <pod-name>` |
| **Execute command in a pod** | `kubectl exec -it <pod-name> -- /bin/sh` |
| **Expose a deployment**     | `kubectl expose deployment <name> --type=LoadBalancer --port=80` |

---

Building a Kubernetes Cluster

Control plane server do:

edit hosts file:
put this in all your nodes (in this example 3 nodes)

## 🔧 Building a Kubernetes Cluster

### Step 1: Configure the Hosts File
Before initializing the cluster, ensure all nodes can resolve each other’s names by editing the `/etc/hosts` file.

On **all nodes** (Control Plane + Workers), add the following:

```sh
sudo vim /etc/hosts
```

```sh
172.31.98.59   k8s-control
172.31.99.1    k8s-worker1
172.31.100.216 k8s-worker2
```


## Step 2: Set Hostnames on All Nodes

Each node needs a unique hostname. Run the following command on each respective node:

### 🖥️ Control Plane Node
```sh
sudo hostnamectl set-hostname k8s-control
```

```sh
sudo hostnamectl set-hostname k8s-worker1
```

```sh
sudo hostnamectl set-hostname k8s-worker2
```

Log out / in

## Step 3: Load Kernel Modules and Configure Networking

Before installing Kubernetes, ensure that the necessary **kernel modules** and **networking settings** are configured.

### 🛠️ Load Required Kernel Modules
Run the following commands to load the required kernel modules:

```sh
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```

Now load the modules into the system:

```sh
sudo modprobe overlay
sudo modprobe br_netfilter
```

---

### ⚙️ Configure Networking Settings
Enable **bridge network traffic forwarding** and **IP forwarding**:

```sh
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```

Apply the changes:

```sh
sudo sysctl --system
```

sudo mkdir /etc/dontainerd

---

💪 **Why is this needed?**
- **`overlay`** module is required for container overlay networks.
- **`br_netfilter`** module allows traffic across pod networks.
- **`sysctl` settings** ensure proper **packet forwarding** for Kubernetes networking.

---

## Step 4: Install Container Runtime
Kubernetes requires a container runtime. Here, we install `containerd`:

```sh
sudo apt-get update && sudo apt-get install -y containerd
```

---

💪 **Why is this needed?**
- **`overlay`** module is required for container overlay networks.
- **`br_netfilter`** module allows traffic across pod networks.
- **`sysctl` settings** ensure proper **packet forwarding** for Kubernetes networking.
- **`containerd`** is the recommended container runtime for Kubernetes.

---

############################################

## Step 3: Load Kernel Modules and Configure Networking

Before installing Kubernetes, ensure that the necessary **kernel modules** and **networking settings** are configured.

### 🛠️ Load Required Kernel Modules
Run the following commands to load the required kernel modules:

```sh
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```

Now load the modules into the system:

```sh
sudo modprobe overlay
sudo modprobe br_netfilter
```

---

### ⚙️ Configure Networking Settings
Enable **bridge network traffic forwarding** and **IP forwarding**:

```sh
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```

Apply the changes:

```sh
sudo sysctl --system
```

---

## Step 5: Disable Swap and Install Dependencies

### Disable Swap
```sh
sudo swapoff -a
```

### Install Required Packages
```sh
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
```

---

## Step 6: Add Kubernetes Repository and Install Packages

### Add Kubernetes GPG Key
```sh
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

### Add Kubernetes Repo
```sh
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

### Update package list
```sh
sudo apt-get update
```

### Install Kubernetes Components (kubelet, kubeadm, kubectl)
```sh
sudo apt install -y kubeadm=1.28.1-1.1 kubelet=1.28.1-1.1 kubectl=1.28.1-1.1
```

### Prevent automatic updates of Kubernetes components
```sh
sudo apt-mark hold kubelet kubeadm kubectl
```

---

## Step 7: Repeat on Worker Nodes
Perform **Steps 3 to 6** on each worker node (`k8s-worker1` and `k8s-worker2`) to ensure they are correctly configured before joining the cluster.

---

💪 **Why is this needed?**
- **Kernel modules** allow necessary networking and container features.
- **`containerd`** is the recommended container runtime for Kubernetes.
- **Disabling swap** is required for stable cluster operations.
- **Installing Kubernetes components** prepares nodes for joining the cluster.

---

## Step 8: Initialize the Kubernetes Cluster
Run the following command on the **control plane node** to initialize the Kubernetes cluster:

```sh
sudo kubeadm init --pod-network-cidr 192.168.0.0/16 --kubernetes-version 1.27.0
```

## Step 9: Set Up kubectl for the Control Plane Node
To start using the cluster, configure `kubectl` for the control plane user:

```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Verify the cluster is running:

```sh
kubectl get nodes
```

```sh
kubectl get nodes
```

At this stage, the control plane node will show a **NotReady** status. This is because a network plugin is not yet installed.

---

## Step 9: Deploy the Network Plugin (Calico)
A **CNI (Container Network Interface) plugin** is required for pod communication. We will use **Calico**:

```sh
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

### Why is this needed?
- **Calico** provides networking and network policies for Kubernetes pods.
- **Kubernetes does not handle networking by itself**, so a CNI plugin is required.
- Applying this manifest **enables inter-pod communication**, allowing the cluster to function correctly.


💪 **Why is this needed?**
- **Kernel modules** allow necessary networking and container features.
- **`containerd`** is the recommended container runtime for Kubernetes.
- **Disabling swap** is required for stable cluster operations.
- **Installing Kubernetes components** prepares nodes for joining the cluster.
- **Initializing the cluster** sets up the control plane and allows nodes to communicate.
- **Applying Calico** enables pod-to-pod networking, making the cluster functional.


---

## Step 10: Join Worker Nodes to the Cluster
On the control plane, generate the join command:

```sh
kubeadm token create --print-join-command
```

This will output a command similar to:

```sh
kubeadm join 172.31.98.59:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

Run this command on **each worker node** (`k8s-worker1` and `k8s-worker2`):

```sh
sudo kubeadm join 172.31.98.59:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

This process will take a few moments as Kubernetes sets up the nodes.

Check the status on the control plane:

```sh
kubectl get nodes
```

Initially, worker nodes may appear as **NotReady**. Wait for them to transition to **Ready**.

```sh
kubectl get nodes
```

Once all nodes are ready, the cluster is fully set up!

---

💪 **Why is this needed?**
- **Worker nodes must join the control plane** to participate in the cluster.
- **Calico networking must be fully applied** before nodes are marked as `Ready`.
- **Verifying node status ensures a functional cluster**.

---

# Using Namespaces in Kubernetes

## What is a Namespace?
A **namespace** in Kubernetes is a logical partition within a cluster that provides a scope for resources. Namespaces help organize and manage resources efficiently, particularly in multi-tenant environments.

Namespaces allow users to:
- Isolate resources between teams, projects, or environments.
- Apply resource quotas and policies at a namespace level.
- Manage access control via Role-Based Access Control (RBAC).

## When to Use Namespaces?
Namespaces are useful when:
- You have multiple teams or applications sharing a cluster.
- You need to enforce resource limits or security policies.
- You want logical separation for development, testing, and production environments.

Namespaces are **not needed** when you have a small setup with few resources, as everything operates in the default namespace.

---

## Working with Namespaces

### List All Namespaces
```sh
kubectl get namespaces
```

### List Pods in a Specific Namespace
```sh
kubectl get pods --namespace your_namespace
```

### List Pods Across All Namespaces
```sh
kubectl get pods --all-namespaces
```

### Create a Namespace
```sh
kubectl create namespace your_namespace
```

Alternatively, using YAML:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: your_namespace
```
Apply it using:
```sh
kubectl apply -f namespace.yaml
```

### Set a Default Namespace for Kubectl Commands
To avoid specifying `--namespace` in every command:
```sh
kubectl config set-context --current --namespace=your_namespace
```

### Delete a Namespace
```sh
kubectl delete namespace your_namespace
```
⚠️ **Note:** Deleting a namespace will remove all resources within it.

### Get Detailed Information About a Namespace
```sh
kubectl describe namespace your_namespace
```

## Conclusion
Namespaces are an essential Kubernetes feature for organizing resources efficiently. By leveraging namespaces, teams can manage workloads more effectively and apply security and resource policies with ease.

