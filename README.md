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