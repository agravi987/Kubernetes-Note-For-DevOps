# 🔧 kubectl — The Kubernetes CLI

> *"Ravi, if Kubernetes is the brain, kubectl is your hands! This is the tool you'll use EVERY. SINGLE. DAY. Let's master it."*

---

## 🤔 What is kubectl?

**kubectl** (pronounced "kube-control" or "kube-cuddle" 😄) is the **command-line tool** to interact with your Kubernetes cluster.

Think of it like a **remote control** for your cluster:
- Want to deploy an app? → `kubectl`
- Want to check if pods are running? → `kubectl`
- Want to debug a crashed container? → `kubectl`
- Want to scale your app from 3 to 100 replicas? → `kubectl`

```bash
# kubectl talks to the API Server
kubectl ──→ API Server ──→ Kubernetes Cluster
```

---

## 💡 Why Do We Need It?

Without kubectl, you'd have to directly call the Kubernetes API with HTTP requests. That's painful!

```bash
# Without kubectl (DON'T DO THIS):
curl -X POST https://api-server:6443/api/v1/namespaces/default/pods \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"apiVersion":"v1","kind":"Pod",...}'

# With kubectl (DO THIS ✅):
kubectl run my-pod --image=nginx
```

kubectl makes everything **simple, readable, and human-friendly**.

---

## 🔄 How It Works

```
┌──────────┐      ┌────────────┐      ┌──────────────┐
│  You     │ ───→ │  kubectl   │ ───→ │  API Server  │
│  (CLI)   │      │            │      │  (K8s)       │
└──────────┘      └────────────┘      └──────────────┘
                         │
                         ↓
               Reads ~/.kube/config
               (cluster address, certs, tokens)
```

kubectl reads your **kubeconfig file** (`~/.kube/config`) to know:
- Which cluster to talk to
- Authentication credentials
- Default namespace

---

## 📚 Essential Commands

### 🏗️ Cluster Information

```bash
# Cluster info (shows API server address, version)
kubectl cluster-info

# Get all nodes in the cluster
kubectl get nodes

# Get nodes with more details
kubectl get nodes -o wide

# API resources (all resource types you can use)
kubectl api-resources

# Cluster version
kubectl version
kubectl version --short
```

---

### 📦 Working with Pods

```bash
# List all pods in current namespace
kubectl get pods

# List pods in all namespaces
kubectl get pods --all-namespaces
kubectl get pods -A

# Get pods with more details (IP, node, etc.)
kubectl get pods -o wide

# Watch pods in real-time (like `tail -f`)
kubectl get pods -w

# Describe a pod (detailed info + events)
kubectl describe pod <pod-name>

# Create a pod
kubectl run my-nginx --image=nginx

# Delete a pod
kubectl delete pod <pod-name>

# Get pod logs
kubectl logs <pod-name>

# Follow logs in real-time
kubectl logs -f <pod-name>

# Logs from a previous crash
kubectl logs <pod-name> --previous

# Execute a command inside a pod
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec -it <pod-name> -- sh    # for Alpine/minimal containers
```

> 😄 **Joke:** Learning kubectl is like learning to use Google Pay — at first you're confused, then you become addicted, and suddenly you can't imagine life without it. 'kubectl get pods' hits different after the 1000th time! 📱

---

### 🚀 Working with Deployments

```bash
# Create a deployment
kubectl create deployment my-app --image=nginx --replicas=3

# List deployments
kubectl get deployments
kubectl get deploy

# Describe a deployment
kubectl describe deployment <name>

# Scale a deployment
kubectl scale deployment my-app --replicas=5

# Update image (rolling update)
kubectl set image deployment/my-app nginx=nginx:1.25

# Rollback a deployment
kubectl rollout undo deployment/my-app

# Check rollout status
kubectl rollout status deployment/my-app

# See rollout history
kubectl rollout history deployment/my-app

# Delete a deployment
kubectl delete deployment <name>
```

---

### 🌐 Working with Services

```bash
# Create a service (expose a deployment)
kubectl expose deployment my-app --type=LoadBalancer --port=80

# List services
kubectl get services
kubectl get svc

# Describe a service
kubectl describe svc <name>

# Delete a service
kubectl delete svc <name>
```

---

### 🗂️ Working with Namespaces

```bash
# List namespaces
kubectl get namespaces

# Create a namespace
kubectl create namespace dev

# Set default namespace (persistent)
kubectl config set-context --current --namespace=dev

# Work in a specific namespace
kubectl get pods -n dev
kubectl get pods --namespace=dev

# List everything in a namespace
kubectl get all -n dev
```

---

### 📝 Applying YAML Files

```bash
# Apply a YAML file (create or update)
kubectl apply -f my-resource.yaml

# Apply a directory of YAML files
kubectl apply -f ./manifests/

# Delete resources from a YAML file
kubectl delete -f my-resource.yaml

# Dry run (validate without applying)
kubectl apply -f my-resource.yaml --dry-run=client

# Force replace
kubectl replace -f my-resource.yaml --force
```

---

## ⚡ Imperative vs Declarative

### Imperative (Quick & Dirty) 🏃
```bash
# Run commands directly — good for learning and quick tests
kubectl run my-pod --image=nginx
kubectl create deployment web --image=nginx --replicas=3
kubectl expose deployment web --port=80 --type=NodePort
```

### Declarative (The Right Way) 📋
```yaml
# my-deployment.yaml — describe what you want
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx
```
```bash
# Apply it
kubectl apply -f my-deployment.yaml
```

> 🎯 **Ravi, always prefer declarative (YAML files) in real projects!** Imperative is great for learning and quick debugging.

---

## 🧠 Important Concepts

### The kubeconfig File

```
~/.kube/config contains:
├── Cluster Info    → API server URL, certificates
├── Users           → Authentication credentials
└── Contexts        → Which cluster + user + namespace to use

# View current context
kubectl config current-context

# List all contexts
kubectl config get-contexts

# Switch context
kubectl config use-context minikube
```

### Output Formats

```bash
# JSON output
kubectl get pods -o json

# YAML output
kubectl get pods -o yaml

# Custom columns
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase

# Table (default)
kubectl get pods -o table
```

### Labels & Selectors with kubectl

```bash
# Filter by label
kubectl get pods -l app=web
kubectl get pods -l 'app in (web, api)'

# Add labels
kubectl label pod my-pod env=production

# Remove labels
kubectl label pod my-pod env-
```

---

## 📋 Best Practices

- ✅ **Use `kubectl apply -f`** instead of `create` for production
- ✅ **Use namespaces** to organize resources
- ✅ **Use `-o wide`** when you need extra info
- ✅ **Use `-w` (watch)** for real-time monitoring
- ✅ **Set up aliases** to save keystrokes:
  ```bash
  # Add to ~/.bashrc or ~/.zshrc
  alias k=kubectl
  alias kgp='kubectl get pods'
  alias kgs='kubectl get svc'
  alias kgn='kubectl get nodes'
  alias kd='kubectl describe'
  alias kaf='kubectl apply -f'
  ```
- ✅ **Use tab completion** — kubectl supports it!
  ```bash
  # Bash
  source <(kubectl completion bash)

  # Zsh
  source <(kubectl completion zsh)
  ```
- ❌ **Don't memorize everything** — use `kubectl --help` and cheat sheets

---

## ❌ Common Mistakes

1. **Forgetting the namespace** 🤦
   ```bash
   # Wrong — might look in wrong namespace
   kubectl get pods

   # Right — specify namespace
   kubectl get pods -n my-namespace
   ```

2. **Using `kubectl exec` without `-it`** 🔒
   ```bash
   # Wrong — can't interact
   kubectl exec my-pod -- bash

   # Right — interactive + TTY
   kubectl exec -it my-pod -- bash
   ```

3. **Confusing `create` and `apply`** 🔄
   ```bash
   # create → fails if resource already exists
   kubectl create -f my-pod.yaml    # ❌ Error on second run

   # apply → creates or updates (idempotent!)
   kubectl apply -f my-pod.yaml     # ✅ Works every time
   ```

4. **Not checking events when debugging** 🔍
   ```bash
   # Always check events!
   kubectl describe pod <pod-name>   # Look at "Events" section
   kubectl get events --sort-by=.metadata.creationTimestamp
   ```

5. **Running kubectl commands without context** 🤷
   ```bash
   # Check which cluster you're talking to:
   kubectl config current-context
   ```

---

## 🎤 Interview Questions

1. **What is kubectl?**
   > The command-line interface tool used to interact with Kubernetes clusters. It communicates with the API Server to create, manage, and delete resources.

2. **What is the difference between `kubectl create` and `kubectl apply`?**
   > `create` fails if the resource already exists. `apply` is idempotent — it creates if it doesn't exist and updates if it does.

3. **How do you get logs from a crashed pod?**
   > `kubectl logs <pod-name> --previous` — retrieves logs from the previous container instance.

4. **What is a kubeconfig file?**
   > A configuration file (`~/.kube/config`) that stores cluster connection details, authentication credentials, and context information.

5. **How do you execute a command inside a running pod?**
   > `kubectl exec -it <pod-name> -- <command>`

6. **What is the difference between `-o wide` and `-o yaml`?**
   > `-o wide` shows extra columns in table format (IP, node). `-o yaml` outputs the full resource definition in YAML format.

---

## 📝 Summary

| Command | What It Does |
|---------|-------------|
| `kubectl get` | List resources |
| `kubectl describe` | Detailed info + events |
| `kubectl apply -f` | Create/update from YAML |
| `kubectl delete` | Remove resources |
| `kubectl logs` | View container logs |
| `kubectl exec -it` | Shell into a container |
| `kubectl scale` | Change replica count |
| `kubectl rollout` | Manage deployments |
| `kubectl config` | Manage kubeconfig |

> *"Ravi, kubectl is your best friend in the Kubernetes world. Use it every day, and it'll become second nature. Pro tip: set up aliases — your fingers will thank you! 🙏"*
