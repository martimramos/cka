# ✅ Certified Kubernetes Administrator (CKA) Cheatsheet

---

## 🐳 Basic Pod Commands

```bash
# Create a simple pod
kubectl run nginx-pod --image=nginx

# Create a pod manifest without launching
kubectl run nginx --image=nginx --dry-run=client -o yaml

# Check image used by a pod
kubectl get pod <pod-name> -o jsonpath="{.spec.containers[*].image}"

# Delete a pod
kubectl delete pod webapp

# Create a pod using a YAML file (even if image is wrong)
vim redis-pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis123  # wrong image intentionally
```

---

## 🛠️ Controllers vs ReplicaSets vs Deployments

* **Pod**: Smallest unit — runs containers.
* **ReplicaSet**: Ensures N pods are running; restarts crashed pods.
* **Deployment**: Manages ReplicaSets; supports updates, rollbacks.

Use **Deployments** in real projects.

---

## 🧠 Kubernetes Controllers Overview

* **ReplicaSet**: Ensures N identical pods
* **Deployment**: Manages ReplicaSets (rolling updates, rollbacks)
* **StatefulSet**: Ordered/stable pods (with identity)
* **DaemonSet**: Runs a pod on every node
* **Job/CronJob**: Run tasks to completion or on a schedule

---

## 📦 ReplicaSet Management

### Create from YAML

```bash
kubectl apply -f /root/replicaset-definition-1.yaml
```

### Example ReplicaSet YAML

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-2
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myimage
```

### View and Edit

```bash
kubectl get rs
kubectl describe rs new-replica-set
kubectl edit rs new-replica-set
```

### Scale ReplicaSet

```bash
# Method 1: scale command
kubectl scale replicaset new-replica-set --replicas=2

# Method 2: edit directly
kubectl edit replicaset new-replica-set
# change replicas: 4 -> 2
```

### Delete ReplicaSet

```bash
kubectl delete replicaset replicaset-1 replicaset-2
```

---

## 🚀 Deployments

### Create

```bash
kubectl create deployment --image=nginx nginx

# with 4 replicas
kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deploy.yaml
kubectl apply -f nginx-deploy.yaml
```

### Inspect

```bash
kubectl get deployments
kubectl describe deployment <deployment-name>
```

### From YAML

```bash
kubectl apply -f /root/deployment-definition-1.yaml
```

**Note:** Ensure kind is capitalized: `kind: Deployment`

### Example Issue

```bash
# Pods stuck in ImagePullBackOff
kubectl describe pod <pod-name>
# Check Events section for image errors
```

---

## 🔍 Troubleshooting & Filtering

### Filter pods by name prefix

```bash
kubectl get pods -o wide | grep ^newpods-
```

### Check which node a pod is on

```bash
kubectl get pods -o wide
```

---

## ✅ Summary

* Use **Deployments** over raw ReplicaSets
* Pods with invalid images show `ImagePullBackOff`
* YAML definitions must have **correct selectors & labels**
* Use `--dry-run=client -o yaml` to safely generate YAML before applying
* Always match `selector.matchLabels` with `template.metadata.labels` in ReplicaSets

---

## 🔗 Kubernetes Services?

A **Kubernetes Service** is a stable virtual IP (ClusterIP) that provides reliable access to a set of Pods, even if they come and go.

---

## 🧩 Service Types

* **ClusterIP** (default): internal access only (inside the cluster)
* **NodePort**: opens a specific port on all Nodes for external access
* **LoadBalancer**: provisioned by cloud providers for external traffic
* **ExternalName**: maps service to a DNS name outside the cluster

---

## 🎯 Why Services?

* Load balance traffic across Pods using labels
* Decouple access from Pod IPs (which change)
* Enable stable discovery and communication

---

## 🧪 Inspecting Services

```bash
kubectl get services
kubectl get service <name> -o yaml
kubectl get endpoints <service-name>
```

### 🧾 Example Output (2 endpoints):

```bash
NAME         ENDPOINTS                        AGE
kubernetes   172.17.0.2:6443,172.17.0.3:6443   25m
```

Each `IP:PORT` = 1 endpoint.

---

## 🛠️ Create a Service from File

```bash
kubectl apply -f service-definition-1.yaml
```

---

## 🔧 Example: ClusterIP Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
```

This routes traffic from port 80 to port 8080 of Pods with label `app=myapp`.

---

## 🌐 Example: NodePort Service 

```yaml
apiVersion: v1               # API version used for the Service object
kind: Service                # Resource type is a Service
metadata:
  name: webapp-service       # Name of the service
  namespace: default         # Namespace where it will be created
spec:
  type: NodePort             # Exposes service on each Node's IP at a static port
  selector:                  # Matches pods with this label
    name: simple-webapp      # Targets pods with label name=simple-webapp
  ports:
    - port: 8080             # Port exposed by the service (inside the cluster)
      targetPort: 8080       # Port on the container to forward to
      nodePort: 30080        # External port on the node (for public access)
```

---

## 🔄 NodePort Flow Diagram

```markdown
User (via browser or curl)
        |
        v
NodeIP:30080  (nodePort)
        |
        v
Service: webapp-service
  - type: NodePort
  - port: 8080
        |
        v
Pod(s) with label: name=simple-webapp
  - containerPort: 8080
```

### 🔄 Flow:

* User accesses `http://<NodeIP>:30080`
* NodePort forwards to Service `port: 8080`
* Service forwards to `targetPort: 8080` on the matching Pods

---

## ✅ Quick Facts

* Default service type is: `ClusterIP`
* To get endpoints: `kubectl get endpoints <svc>`
* Create services with YAML and `kubectl apply -f`
* Each endpoint = IP\:Port combo linked to the selected Pods
* 

# Namespaces, Quotas & Service DNS

---

## 🧭 Kubernetes Namespaces

A **Namespace** is like a **folder** for your Kubernetes resources.

---

### ✅ Purpose

* Organize resources (pods, services, etc.)
* Isolate environments (e.g., `dev`, `test`, `prod`)
* Enable multi-team or multi-tenant clusters
* Apply resource limits and RBAC rules

---

### 📦 Default Namespaces

* `default`: standard workspace
* `kube-system`: core system components
* `kube-public`: readable by all users
* `kube-node-lease`: for node heartbeat tracking

---

### 🔧 Namespace Commands

```bash
kubectl get namespaces
kubectl create namespace my-namespace
kubectl delete namespace my-namespace
kubectl get pods -n my-namespace

# Accessing pods in specific namespaces
kubectl get pods --namespace=dev
kubectl get pods --namespace=prod
kubectl get pods --namespace=default
kubectl get pods
kubectl get pods --all-namespaces

# Change default namespace for the current context
kubectl config set-context $(kubectl config current-context) --namespace=dev
kubectl config set-context $(kubectl config current-context) --namespace=prod
```

Use `-n` or `--namespace` to target a specific namespace.

---

## 🧮 Kubernetes Resource Quota – Explained Simply

A **ResourceQuota** restricts the total CPU, memory, and pod count usage **per namespace**.

### 📌 Purpose

* Prevent resource hogging
* Enforce fair sharing
* Ensure all teams declare resource limits in specs

---

### 🧱 ResourceQuota Example YAML

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota       # Quota name
  namespace: dev            # Applies to the 'dev' namespace
spec:
  hard:
    pods: "10"              # Max 10 pods allowed
    requests.cpu: "4"       # Total CPU requested ≤ 4 cores
    requests.memory: 5Gi    # Total memory requested ≤ 5Gi
    limits.cpu: "10"        # Max CPU limit set ≤ 10 cores
    limits.memory: 10Gi     # Max memory limit set ≤ 10Gi
```

---

### ✅ Apply the Quota

```bash
kubectl create -f compute-quota.yaml
```

### 🖼️ Quota Behavior Summary

* Quotas apply at the namespace level
* Resource usage across all nodes is aggregated per namespace
* Nodes are not limited, just the namespace usage totals

---

## 🌐 Kubernetes Service DNS Access

### 🔎 Question: What DNS name should the Blue app use to access the `db-service` in its own namespace `marketing`?

### ✅ Answer:

* **Short DNS** (within same namespace):

  ```
  db-service
  ```
* **Full DNS** (any namespace):

  ```
  db-service.marketing.svc.cluster.local
  ```

Use the short name when the client pod is in the **same namespace**.


