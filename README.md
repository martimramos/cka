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

# High Availability in Kubernetes (K8s)

## Introduction
High Availability (HA) in Kubernetes ensures that both applications and the cluster itself remain operational even in the event of failures. By designing for HA, organizations can minimize downtime and maintain service reliability.

To achieve high availability, Kubernetes clusters typically use multiple control plane nodes, load balancing, and redundancy mechanisms.

---

## High Availability Control Plane

### Why Multiple Control Planes?
Kubernetes relies on the **control plane** to manage cluster operations. If the control plane goes down, workloads may still run, but new workloads cannot be scheduled. A highly available Kubernetes cluster requires multiple control plane nodes to avoid single points of failure.

### Architecture
- Multiple **control plane nodes** running `kube-api-server`.
- A **load balancer** to distribute requests to different control plane nodes.
- Worker nodes communicate with the control plane through the load balancer.

#### Example Architecture:
```
+-------------------------+      +-------------------------+
|  Control Plane Node 1   |      |  Control Plane Node 2   |
|  --------------------   |      |  --------------------   |
|  kube-api-server        |      |  kube-api-server        |
+-------------------------+      +-------------------------+
        \                          /
         \                        /
        +----------------------+
        |    Load Balancer     |
        +----------------------+
                |
        +------------------+
        |    Worker Node   |
        |    kubelet       |
        +------------------+
```
### Key Benefits:
- Ensures **API server availability** through redundancy.
- Allows **failover** between control plane nodes.
- Enables **scalability** by distributing workloads across multiple nodes.

---

## etcd Configuration for HA
`etcd` is a key component of Kubernetes, acting as the cluster's backing store. There are two main deployment models for `etcd` in an HA setup:

### 1. **Stacked etcd**
- Each control plane node runs its own `etcd` instance.
- The `etcd` cluster is formed by all control plane nodes.
- Easier to deploy, but failure of multiple nodes can cause data loss.

#### Example:
```
+-------------------------+      +-------------------------+
|  Control Plane Node 1   |      |  Control Plane Node 2   |
|  --------------------   |      |  --------------------   |
|  kube-api-server        |      |  kube-api-server        |
|  etcd                   |      |  etcd                   |
+-------------------------+      +-------------------------+
        |                              |
        +------------+----------------+
                     |
                  etcd cluster
```

### 2. **External etcd**
- `etcd` runs on dedicated nodes outside of the control plane.
- This provides better failure tolerance and scalability.
- More complex to set up but recommended for production-grade clusters.

#### Example:
```
+-------------------------+      +-------------------------+
|  Control Plane Node 1   |      |  Control Plane Node 2   |
|  --------------------   |      |  --------------------   |
|  kube-api-server        |      |  kube-api-server        |
+-------------------------+      +-------------------------+
        \                          /
         \                        /
        +----------------------+
        |   External etcd      |
        +----------------------+
```

---

## Key Components of HA in Kubernetes

### 1. **Load Balancer**
- Distributes traffic among control plane nodes.
- Can be an external load balancer (e.g., HAProxy, Nginx, AWS ELB) or internal Kubernetes component.

### 2. **Multiple Control Plane Nodes**
- Ensures redundancy in case of a node failure.
- Avoids single points of failure in the cluster.

### 3. **Redundant etcd**
- Stores the cluster state and configurations.
- Running it in HA mode prevents data loss and ensures consistency.

### 4. **Worker Nodes with kubelet**
- Continue running workloads even if the control plane is temporarily unavailable.
- Communicate with the API server through the load balancer.

---

## Conclusion
High availability in Kubernetes is essential for ensuring resilience, reliability, and uptime in production environments. By implementing multiple control plane nodes, a load balancer, and a highly available `etcd` setup, organizations can mitigate risks and ensure smooth cluster operations.

### Additional Considerations:
- Use **node affinity and anti-affinity** rules to distribute workloads evenly.
- Regularly **back up etcd** to prevent data loss.
- Monitor control plane health using **Prometheus, Grafana, or Kubernetes metrics-server**.

By following these best practices, you can build a fault-tolerant Kubernetes cluster ready for high-availability demands.

# Kubernetes Management Tools

## Introduction
Kubernetes management tools help users interact with, deploy, and configure Kubernetes clusters more efficiently. These tools simplify cluster management, deployment automation, and configuration handling.

## Common Kubernetes Management Tools

### 1. **kubectl**
- Official CLI tool for Kubernetes.
- Used for managing and interacting with Kubernetes clusters.
- Essential for daily operations and the CKA exam.

### 2. **kubeadm**
- Simplifies the setup of Kubernetes clusters.
- Helps initialize the control plane and join worker nodes.

### 3. **Minikube**
- Creates a single-node Kubernetes cluster on a local machine.
- Ideal for development and testing.

### 4. **Helm**
- Kubernetes package manager.
- Uses charts to manage and deploy complex applications.

### 5. **Kompose**
- Converts Docker Compose files into Kubernetes manifests.
- Useful for transitioning from Docker to Kubernetes.

### 6. **Kustomize**
- Manages Kubernetes configurations without modifying the base files.
- Allows configuration reuse and customization.

## Conclusion
These tools enhance Kubernetes usability by simplifying cluster setup, configuration, and application management. Understanding and using them effectively can improve efficiency in Kubernetes operations.

# Safely Draining a Kubernetes Node

## Introduction
Draining a Kubernetes node is essential when performing maintenance to ensure that workloads continue running without interruption. This process gracefully evicts pods from a node while ensuring that critical applications remain available.

## What is Draining?
Draining a node removes it from service, gracefully terminating running pods and rescheduling them onto other nodes. This ensures uninterrupted service during maintenance.

## Draining a Node
Use the following command to drain a node:
```sh
kubectl drain <node-name>
```

However, this may fail if there are DaemonSet-managed pods or standalone pods that are not managed by a controller.

### Ignoring DaemonSets
DaemonSets cannot be rescheduled, so they must be ignored when draining a node:
```sh
kubectl drain <node-name> --ignore-daemonsets
```

### Forcing Pod Eviction
To remove non-managed standalone pods, use the `--force` flag:
```sh
kubectl drain <node-name> --ignore-daemonsets --force
```
⚠️ **Warning:** Using `--force` will delete standalone pods permanently.

## Uncordoning a Node
Once maintenance is complete, uncordoning a node allows Kubernetes to schedule workloads on it again:
```sh
kubectl uncordon <node-name>
```

## Key Takeaways
- Draining a node moves workloads gracefully to prevent downtime.
- Use `--ignore-daemonsets` to handle system-critical pods.
- `--force` can be used to remove unmanaged pods but should be used cautiously.
- Uncordoning a node restores it for scheduling but does not automatically rebalance workloads.

Understanding `kubectl drain` and `kubectl uncordon` helps ensure a smooth maintenance process in a Kubernetes cluster.

# Upgrading Kubernetes with kubeadm

## Introduction
Upgrading a Kubernetes cluster is essential to stay up to date with security patches, performance improvements, and new features. `kubeadm` simplifies the upgrade process while ensuring minimal downtime.

## Overview of the Upgrade Process
Upgrading a Kubernetes cluster consists of two main steps:
1. **Upgrading the Control Plane**
2. **Upgrading the Worker Nodes**

The upgrade must be performed **one node at a time** to ensure minimal service disruption.

---

## Upgrading the Control Plane
1. **Drain the control plane node**
   ```sh
   kubectl drain <control-plane-node> --ignore-daemonsets --force
   ```
2. **Upgrade kubeadm**
   ```sh
   sudo apt-get update && sudo apt-get install -y kubeadm=1.27.2 --allow-change-held-packages
   ```
3. **Plan the upgrade**
   ```sh
   sudo kubeadm upgrade plan
   ```
4. **Apply the upgrade**
   ```sh
   sudo kubeadm upgrade apply 1.27.2
   ```
5. **Upgrade kubelet and kubectl**
   ```sh
   sudo apt-get install -y kubelet=1.27.2 kubectl=1.27.2 --allow-change-held-packages
   ```
6. **Restart kubelet**
   ```sh
   sudo systemctl daemon-reload
   sudo systemctl restart kubelet
   ```
7. **Uncordon the control plane node**
   ```sh
   kubectl uncordon <control-plane-node>
   ```
8. **Verify the upgrade**
   ```sh
   kubectl get nodes
   ```

---

## Upgrading Worker Nodes
Repeat the following steps **for each worker node**, one at a time:
1. **Drain the worker node**
   ```sh
   kubectl drain <worker-node> --ignore-daemonsets --force
   ```
2. **Upgrade kubeadm**
   ```sh
   sudo apt-get update && sudo apt-get install -y kubeadm=1.27.2 --allow-change-held-packages
   ```
3. **Apply the node upgrade**
   ```sh
   sudo kubeadm upgrade node
   ```
4. **Upgrade kubelet and kubectl**
   ```sh
   sudo apt-get install -y kubelet=1.27.2 kubectl=1.27.2 --allow-change-held-packages
   ```
5. **Restart kubelet**
   ```sh
   sudo systemctl daemon-reload
   sudo systemctl restart kubelet
   ```
6. **Uncordon the worker node**
   ```sh
   kubectl uncordon <worker-node>
   ```
7. **Verify the upgrade**
   ```sh
   kubectl get nodes
   ```

---

## Conclusion
- Kubernetes upgrades should be performed **one node at a time** to minimize downtime.
- The process involves upgrading `kubeadm`, running `kubeadm upgrade`, and updating `kubelet` and `kubectl`.
- After upgrading, verify all nodes are running the new version using `kubectl get nodes`.

By following these steps, you can safely and efficiently upgrade your Kubernetes cluster using `kubeadm`.

# Backing Up and Restoring etcd in Kubernetes

## Introduction
`etcd` is the key-value store that holds the entire state of a Kubernetes cluster, including configurations, objects, and application metadata. Backing up `etcd` is essential to prevent data loss and allow cluster recovery in case of failure.

---

## Why Back Up etcd?
- `etcd` contains the **entire configuration and state** of a Kubernetes cluster.
- Losing `etcd` data means **losing the cluster configuration**, requiring manual reconstruction.
- Regular backups ensure that Kubernetes objects and applications can be **restored** quickly in case of failure.

---

## Backing Up etcd
Use the `etcdctl` CLI tool to create a snapshot of `etcd` data.

### Backup Command:
```sh
ETCDCTL_API=3 etcdctl snapshot save /path/to/backup.db \
  --endpoints=https://10.0.1.101:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

### Reset etcd by removing all existing etcd data:
```sh
sudo systemctl stop etcd
sudo rm -rf /var/lib/etcd
```

### Explanation:
- `snapshot save` creates a backup file.
- `--endpoints` specifies the `etcd` API server address.
- `--cacert`, `--cert`, and `--key` authenticate the request using Kubernetes certificates.

---

## Restoring etcd
Restoring `etcd` from a backup requires creating a **new etcd cluster instance** and loading the saved snapshot.

### Restore Command:
```sh
sudo ETCDCTL_API=3 etcdctl snapshot restore /path/to/backup.db
  --initial-cluster etcd-restore=https://10.0.1.101:2380 \
  --initial-advertise-peer-urls https://10.0.1.101:2380 \
  --name etcd-restore \
  --data-dir /var/lib/etcd
```
```sh
sudo chown -R etcd:etcd /var/lib/etcd
sudo systemctl start etcd
```
### Additional Steps:
1. **Update etcd configuration** to point to the new `data-dir`.
2. **Restart etcd** to apply the restored data.
3. **Verify cluster health** with:
   ```sh
   ETCDCTL_API=3 etcdctl endpoint health
   ```

---

## Conclusion
- Backing up `etcd` regularly ensures Kubernetes cluster **recovery** in case of failures.
- Use `etcdctl snapshot save` to create a **backup**.
- Use `etcdctl snapshot restore` to **recover** data if needed.
- Always **verify cluster health** after restoring.

By practicing these steps, you can effectively safeguard your Kubernetes cluster data against accidental loss or corruption.
