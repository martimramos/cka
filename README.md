# CKA Cheat Sheet

## Certified Kubernetes Administrator (CKA) - Cheat Sheet

**Certification:** [Certified Kubernetes Administrator (CKA)](https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/)  
**Purpose:** Master Kubernetes cluster management, troubleshooting, networking, and security.

---

## 🚀 **Container Orchestration**
Kubernetes' primary role is to dynamically manage containers across multiple hosts, ensuring efficient resource utilization and application deployment.

## 🔄 **Application Reliability**
Kubernetes simplifies the creation of **reliable, self-healing, and scalable** applications by managing replicas, health checks, and rolling updates.

## ⚙️ **Automation**
Kubernetes provides **declarative configuration and automation**, making it easier to manage containerized applications with features like:
- **Auto-scaling** (HPA, VPA)
- **Rolling updates & rollbacks**
- **Self-healing pods**
- **Declarative state management (YAML)**

---

## 🏗️ **Kubernetes Control Plane**
The **Control Plane** is responsible for managing the cluster and ensuring desired state enforcement. It includes:
- **kube-apiserver** – Central communication hub for all Kubernetes components.
- **etcd** – Distributed key-value store for cluster state.
- **kube-scheduler** – Assigns workloads (Pods) to Nodes.
- **kube-controller-manager** – Handles controllers like replication, node lifecycle, and service accounts.
- **cloud-controller-manager** – Manages cloud provider integrations.


```mermaid
graph TD;
    subgraph "Control Plane"
        KubeAPIServer["kube-api-server"]
        Etcd["Etcd"]
        KubeControllerManager["kube-controller-manager"]
        KubeScheduler["kube-scheduler"]
        CloudControllerManager["cloud-controller-manager"]

        KubeAPIServer -->|Stores state| Etcd
        KubeControllerManager -->|Communicates with| KubeAPIServer
        KubeScheduler -->|Communicates with| KubeAPIServer
        CloudControllerManager -->|Communicates with| KubeAPIServer
    end

    subgraph "Nodes"
        subgraph "Node 1"
            KubeProxy1["kube-proxy"]
            Kubelet1["kubelet"]
            ContainerRuntime1["container runtime"]
            Container1["container"]
            Container2["container"]
            Container3["container"]
            Container4["container"]
            KubeProxy1 --> Kubelet1
            Kubelet1 --> ContainerRuntime1
            ContainerRuntime1 --> Container1
            ContainerRuntime1 --> Container2
            ContainerRuntime1 --> Container3
            ContainerRuntime1 --> Container4
        end

        subgraph "Node 2"
            KubeProxy2["kube-proxy"]
            Kubelet2["kubelet"]
            ContainerRuntime2["container runtime"]
            Container5["container"]
            Container6["container"]
            Container7["container"]
            Container8["container"]
            KubeProxy2 --> Kubelet2
            Kubelet2 --> ContainerRuntime2
            ContainerRuntime2 --> Container5
            ContainerRuntime2 --> Container6
            ContainerRuntime2 --> Container7
            ContainerRuntime2 --> Container8
        end

        subgraph "Node 3"
            KubeProxy3["kube-proxy"]
            Kubelet3["kubelet"]
            ContainerRuntime3["container runtime"]
            Container9["container"]
            Container10["container"]
            Container11["container"]
            Container12["container"]
            KubeProxy3 --> Kubelet3
            Kubelet3 --> ContainerRuntime3
            ContainerRuntime3 --> Container9
            ContainerRuntime3 --> Container10
            ContainerRuntime3 --> Container11
            ContainerRuntime3 --> Container12
        end
    end

    KubeAPIServer -->|Manages nodes| KubeProxy1
    KubeAPIServer -->|Manages nodes| KubeProxy2
    KubeAPIServer -->|Manages nodes| KubeProxy3



