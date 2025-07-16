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
