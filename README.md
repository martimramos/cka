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


## 🆚 Imperative vs Declarative in Kubernetes

### Imperative:

* You tell Kubernetes *what to do now* using CLI.
* Fast, direct, but not easily repeatable or versioned.

### Declarative:

* You describe the *desired state* in a YAML file.
* Great for version control, automation, and repeatability.

---

## 🛠️ Common Imperative Commands

```bash
kubectl run nginx --image=nginx                           # Run a pod
kubectl create deployment nginx --image=nginx             # Create a deployment
kubectl expose deployment nginx --port=80                 # Expose service
kubectl edit deployment nginx                             # Edit live object
kubectl scale deployment nginx --replicas=5               # Scale replicas
kubectl set image deployment nginx nginx=nginx:1.18       # Update image
```

---

## 📄 Declarative YAML Usage

```bash
kubectl create -f nginx.yaml        # Create from YAML
kubectl replace -f nginx.yaml       # Replace
kubectl delete -f nginx.yaml        # Delete
```

### Example YAML (`nginx.yaml`):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end
spec:
  containers:
  - name: nginx-container
    image: nginx
```

---

## 🔁 After Live Edit (example `kubectl edit` output)

```yaml
...
spec:
  containers:
  - name: nginx-container
    image: nginx:1.18
...
```

---

## ✅ Summary Table

| Type        | Command Example               | Stored In    | Best For             |
| ----------- | ----------------------------- | ------------ | -------------------- |
| Imperative  | `kubectl run`, `kubectl edit` | In-memory    | Quick fixes, testing |
| Declarative | `kubectl apply -f file.yaml`  | Source files | CI/CD, automation    |

---

## 🧠 Tips for YAML via CLI

* Use `--dry-run=client -o yaml` to generate resource definitions without creating them:

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx.yaml
```

Modify and apply:

```bash
kubectl apply -f nginx.yaml
```

---

## 🔧 Pod & Deployment Examples

### Pod (Imperative)

```bash
kubectl run nginx --image=nginx
```

### Pod (YAML Generation)

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

### Deployment (Imperative)

```bash
kubectl create deployment nginx --image=nginx
kubectl scale deployment nginx --replicas=4
```

### Deployment (YAML to File)

```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

---

## 🔌 Services (Imperative & Declarative)

### Create ClusterIP for Redis Pod

```bash
kubectl expose pod redis --port=6379 --name=redis-service --type=ClusterIP
```

### NodePort Example (exposing nginx)

```bash
kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service
```

> To specify a nodePort, generate the YAML first, then edit:

```bash
kubectl expose pod nginx --type=NodePort --port=80 --dry-run=client -o yaml > svc.yaml
```

Add:

```yaml
nodePort: 30080
```

Then:

```bash
kubectl apply -f svc.yaml
```

---

## 🔨 Quick Commands Recap

### Create Pod

```bash
kubectl run nginx-pod --image=nginx:alpine
```

### Pod with Labels

```bash
kubectl run redis --image=redis:alpine --labels=tier=db
```

### Generate YAML & Edit Labels

```bash
kubectl run redis --image=redis:alpine --dry-run=client -o yaml > redis.yaml
# edit file to add:
# metadata.labels.tier: db
kubectl apply -f redis.yaml
```

### Create Service from Pod

```bash
kubectl expose pod redis --name=redis-service --port=6379 --target-port=6379 --type=ClusterIP
```

### Create Deployment with Replicas

```bash
kubectl create deployment webapp --image=kodekloud/webapp-color --replicas=3
```

### Create Namespace

```bash
kubectl create namespace dev-ns
```

### Deployment in a Namespace

```bash
kubectl create deployment redis-deploy --image=redis --replicas=2 -n dev-ns
```

### Pod + Service in 2 Steps

```bash
kubectl run httpd --image=httpd:alpine --port=80
kubectl expose pod httpd --port=80 --target-port=80 --name=httpd --type=ClusterIP
```

---

## 📚 References

* [Kubectl Official Docs](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)
* [Kubectl Conventions](https://kubernetes.io/docs/reference/kubectl/conventions/)

---

Imperative is great for speed and experimentation. Declarative is essential for automation, consistency, and production-grade deployments.


## ✅ Kubectl Apply – How it Works

When you use `kubectl apply -f file.yaml`, Kubernetes performs a **3-way merge** between:

1. **Your local file** – the new desired state (e.g. `nginx.yaml`)
2. **Last-applied-configuration** – what you applied last time (stored as annotation in the live object)
3. **Live object configuration** – the current state of the object in the cluster

---

### 📄 Local File (new intent)

This is the YAML you apply:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: nginx-container
    image: nginx:1.19
```

---

### 🔁 Last-Applied Configuration

This is stored as an annotation in the live object under:

```yaml
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: '{ ... json ... }'
```

This helps `kubectl` detect what changed between the last apply and the current YAML.

---

### 📦 Live Object Configuration

This is what actually runs in the cluster. It may have been changed manually or by controllers:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: nginx-container
    image: nginx:1.19
status:
  conditions:
  - type: Initialized
    status: "True"
```

---

### 🔄 Apply Process Summary

1. Compare **your YAML** with the **last-applied-configuration**.
2. Detect changes.
3. Patch only the **differences** into the **live object**.
4. Update the annotation with the new `last-applied-configuration`.

---

### 🧠 Why This Matters

* Manual changes to the live object not reflected in YAML will be **overwritten**.
* Always re-apply using updated YAMLs to ensure drift doesn't happen.
* For safer updates: avoid editing live objects manually (e.g., with `kubectl edit`).

---

### ✅ Best Practices

* Always use `kubectl apply` for declarative workflows.
* Use `kubectl diff -f file.yaml` to preview changes.
* If needed, use `kubectl replace` for full overwrite (not recommended for live apps).

---

### 📌 Command Summary

```bash
kubectl apply -f my.yaml               # Apply declarative config
kubectl get pod myapp-pod -o yaml      # View live object
kubectl annotate ...                   # View/edit annotations
```

---

### 🧩 Tip: View Last Applied Config

```bash
kubectl get pod myapp-pod -o json | jq -r '.metadata.annotations["kubectl.kubernetes.io/last-applied-configuration"]' | jq .
```

This will pretty-print the last-applied config from the annotation (in JSON).

---

By understanding the **3-way diff** and how `kubectl apply` works, you can safely manage your YAML-driven configurations with confidence. 💡

## 🧪 Imperative Commands

Imperative commands are direct `kubectl` commands used to create, update, or delete Kubernetes resources without YAML manifests. Great for quick testing and small tasks.

---

### 🔹 Create Pods
```bash
kubectl run <pod-name> --image=<image-name>
# Example:
kubectl run nginx-pod --image=nginx:alpine
```

### 🔹 Add Labels When Creating
```bash
kubectl run <pod-name> --image=<image-name> --labels=key=value
# Example:
kubectl run redis --image=redis:alpine --labels=tier=db
```

### 🔹 Create Services
```bash
kubectl expose pod <pod-name> --port=<port> --name=<service-name>
# Example:
kubectl expose pod redis --port=6379 --name=redis-service
```

### 🔹 Create Deployments
```bash
kubectl create deployment <name> --image=<image-name> --replicas=<n>
# Example:
kubectl create deployment webapp --image=kodekloud/webapp-color --replicas=3
```

### 🔹 Scale Deployments (if --replicas not accepted initially)
```bash
kubectl scale deployment <name> --replicas=<n>
```

### 🔹 Generate YAML Without Creating (Dry Run)
```bash
kubectl run <pod-name> --image=<image-name> --dry-run=client -o yaml > pod.yaml
# Edit pod.yaml as needed, e.g., add containerPort, labels, etc.
kubectl apply -f pod.yaml
```

### 🔹 Create Namespace
```bash
kubectl create namespace <namespace-name>
# Example:
kubectl create namespace dev-ns
```

### 🔹 Create Deployment in Specific Namespace
```bash
kubectl create deployment <name> --image=<image-name> --replicas=<n> -n <namespace>
# Example:
kubectl create deployment redis-deploy --image=redis --replicas=2 -n dev-ns
```

### 🔹 Verify Resources
```bash
kubectl get pods
kubectl get svc
kubectl get deployments
kubectl get namespaces
kubectl get pods -n <namespace>
```

### 🔹 Describe Resources
```bash
kubectl describe pod <pod-name>
kubectl describe svc <service-name>
```

### 🔹 Delete Resources
```bash
kubectl delete pod <pod-name>
kubectl delete deployment <name>
kubectl delete svc <name>
kubectl delete namespace <namespace-name>



## 🧭 Scheduling (CKA Cheatsheet)

This section covers Kubernetes pod scheduling essentials, especially useful for CKA exam scenarios.

---

### 🔍 Investigating Pending Pods

```bash
kubectl describe pod <pod-name>
```
- Check for `Node: <none>` → unscheduled
- Look for events at the bottom (taints, no resources, etc.)

```bash
kubectl get nodes
kubectl describe node <node-name>
```
- Confirm node readiness and check for taints

```bash
kubectl get pods -n kube-system
```
- Ensure all control plane components (especially `kube-scheduler`) are running

---

### ❌ Pod in Pending: Common Causes

- 🚫 **Missing scheduler** → `kube-scheduler` not running
- 🚫 **Taints** on nodes → no matching tolerations
- 🚫 **No nodes available** → all unschedulable
- 🚫 **Resource requests too high** → can't fit on any node
- ⚠️ **NodeSelector/Affinity mismatch**

---

### ✅ Manual Scheduling with `nodeName`

1. **Assign pod to a specific node**:

```yaml
spec:
  nodeName: node01
```

2. **Apply YAML**:
```bash
kubectl apply -f nginx.yaml
```

3. **Check placement**:
```bash
kubectl get pods -o wide
```

---

### ⚙️ Scheduling on a Tainted Node

To schedule on the control-plane node (typically tainted with `NoSchedule`):

```yaml
spec:
  nodeName: controlplane
  tolerations:
    - key: "node-role.kubernetes.io/control-plane"
      operator: "Exists"
      effect: "NoSchedule"
```

Apply and verify:
```bash
kubectl delete pod nginx
kubectl apply -f nginx.yaml
kubectl get pods -o wide
```

---

### 🧪 Extra Useful Commands

```bash
kubectl get pods -A                     # All namespaces
kubectl describe node <node>           # See taints, allocatable resources
kubectl get events --sort-by=.metadata.creationTimestamp  # View recent events
```

---

✅ **Tip for exam**: If a pod is `Pending` with no events, check if the scheduler is even running.

---

