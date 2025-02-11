# CKA Cheat Sheet

## Certified Kubernetes Administrator (CKA) - Cheat Sheet

**Certification:** [Certified Kubernetes Administrator (CKA)](https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/)  
**Purpose:** Master Kubernetes cluster management, troubleshooting, networking, and security.

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
The **Control Plane** manages the cluster and ensures the desired state.

| Component                 | Description |
|---------------------------|------------|
| **kube-apiserver**        | Central API gateway for communication. |
| **etcd**                  | Distributed key-value store for cluster state. |
| **kube-scheduler**        | Assigns workloads (Pods) to Nodes. |
| **kube-controller-manager** | Manages controllers (node lifecycle, replication, etc.). |
| **cloud-controller-manager** | Integrates Kubernetes with cloud providers. |

---

## 🖥️ Kubernetes Node Components
Each **Node** runs application workloads.

| Component          | Description |
|--------------------|------------|
| **kubelet**       | Agent that runs on each node to manage pods. |
| **kube-proxy**    | Manages networking for pod communication. |
| **Container runtime** | Runs the actual containers (Docker, containerd, CRI-O). |

---

## 📌 Additional Key Concepts

| Concept               | Description |
|-----------------------|------------|
| **Pod**              | The smallest deployable unit in Kubernetes. |
| **Deployment**       | Ensures the desired number of pod replicas are running. |
| **Service**         | Exposes applications inside or outside the cluster. |
| **ConfigMap & Secret** | Stores configuration and sensitive data. |
| **PersistentVolume (PV)** | Manages storage for stateful applications. |

---

This version **removes Mermaid diagrams** and instead **uses Markdown tables** for structured information. ✅  
It’s now **fully compatible** with GitHub, GitLab, VS Code, and Markdown preview tools. 🚀

Would you like me to **add YAML command examples** or keep it simple?
