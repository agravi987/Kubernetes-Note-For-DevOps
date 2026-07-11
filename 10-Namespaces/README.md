# 🗂️ Namespaces

> *"Ravi, imagine a big office building with 100 companies. Without walls and floors, it'd be chaos! Namespaces are those walls — they divide your cluster into virtual sections so teams, projects, and environments don't collide. Organization = Sanity! 🧘"*

---

## 🤔 What is a Namespace?

A **Namespace** is a virtual cluster within a physical cluster. It provides a **scope for names** — resources within a namespace must be unique, but names can repeat across namespaces.

```
┌─────────── Physical Cluster ──────────────┐
│                                           │
│  ┌──── Namespace: production ──────────┐  │
│  │  ├── Pod: web-abc                  │  │
│  │  ├── Service: web-svc              │  │
│  │  └── ConfigMap: web-config         │  │
│  └────────────────────────────────────┘  │
│                                           │
│  ┌──── Namespace: staging ────────────┐  │
│  │  ├── Pod: web-abc                  │  │  ← Same name, different namespace!
│  │  ├── Service: web-svc              │  │
│  │  └── ConfigMap: web-config         │  │
│  └────────────────────────────────────┘  │
│                                           │
│  ┌──── Namespace: monitoring ─────────┐  │
│  │  ├── Pod: prometheus               │  │
│  │  └── Service: grafana              │  │
│  └────────────────────────────────────┘  │
└───────────────────────────────────────────┘
```

---

## 💡 Why Do We Need Namespaces?

| Without Namespaces | With Namespaces |
|-------------------|----------------|
| Everything in one flat space 😵 | Organized into logical groups 📁 |
| Name conflicts possible 😱 | Same name OK in different namespaces ✅ |
| No access control | RBAC per namespace 🔐 |
| No resource limits per group | ResourceQuotas per namespace 💰 |
| Hard to manage large clusters | Easy team/project separation 🎯 |

---

## 📚 Default Namespaces

Kubernetes comes with these built-in namespaces:

```bash
kubectl get namespaces

# NAME              STATUS   AGE
# default           Active   5d     ← Your stuff goes here by default
# kube-system       Active   5d     ← K8s system components
# kube-public       Active   5d     ← Publicly readable (even unauthenticated)
# kube-node-lease   Active   5d     ← Node heartbeat data
```

| Namespace | Purpose |
|-----------|---------|
| **default** | Where resources go if you don't specify a namespace |
| **kube-system** | Control plane components (API server, etcd, scheduler) |
| **kube-public** | Public cluster information (readable by everyone) |
| **kube-node-lease** | Node heartbeat/lease data for availability |

---

## 📝 YAML Example

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: development
  labels:
    env: dev
    team: backend
```

```yaml
# A Deployment inside the development namespace
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: development       # ← Specify namespace here
  labels:
    app: web
spec:
  replicas: 2
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
          image: nginx:1.25
```

---

## 📚 kubectl Commands

```bash
# Create a namespace
kubectl create namespace dev

# Apply resources to a namespace
kubectl apply -f deployment.yaml -n dev
kubectl apply -f deployment.yaml --namespace=dev

# List resources in a namespace
kubectl get pods -n dev
kubectl get all -n dev

# List resources in ALL namespaces
kubectl get pods --all-namespaces
kubectl get pods -A

# Set default namespace (persists)
kubectl config set-context --current --namespace=dev

# Check current namespace
kubectl config view --minify | grep namespace

# Delete a namespace (deletes ALL resources inside!)
kubectl delete namespace dev
```

---

## 🧠 Important Concepts

### Namespace Isolation

```
Namespaces provide:
├── Name scoping    → "web" in dev ≠ "web" in prod
├── Access control  → RBAC rules per namespace
├── Resource limits → ResourceQuotas per namespace
└── NetworkPolicy   → Can restrict traffic between namespaces
```

> ⚠️ **Namespaces are NOT security boundaries by default!** Pods in different namespaces can still communicate. Use NetworkPolicies for actual isolation.

### Namespace Communication

```bash
# Access a service in another namespace using DNS
# Format: <service-name>.<namespace>.svc.cluster.local

# From default namespace, reach web-service in production:
curl http://web-service.production.svc.cluster.local

# Short form (within same cluster):
curl http://web-service.production
```

### ResourceQuotas (Per-Namespace Limits)

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: development
spec:
  hard:
    pods: "10"                    # Max 10 pods
    requests.cpu: "2"             # Max 2 CPU cores total
    requests.memory: "4Gi"        # Max 4Gi memory total
    limits.cpu: "4"               # Max 4 CPU cores total
    limits.memory: "8Gi"          # Max 8Gi memory total
```

> 🏢 **Fun Fact / Joke:** Namespaces are like Indian society flats — everyone's in the same building (cluster), but each flat (namespace) has its own rules, water timings, and that one aunty who monitors everything. 🏢👀

---

## 📋 Best Practices

- ✅ **Use namespaces** for separating environments (dev, staging, prod)
- ✅ **Use namespaces** for multi-team clusters
- ✅ **Set ResourceQuotas** on each namespace to prevent resource abuse
- ✅ **Set default resource limits** with LimitRange per namespace
- ✅ **Use RBAC** to control who can access which namespace
- ❌ **Don't create too many namespaces** — they add complexity
- ❌ **Don't rely on namespaces for security** — use NetworkPolicies
- ❌ **Don't use default namespace** — always create your own
- ❌ **Don't delete a namespace** without understanding it deletes everything inside

---

## ❌ Common Mistakes

1. **Using the `default` namespace** 🤦
   > Always create and use your own namespaces. The default namespace becomes a junk drawer.

2. **Forgetting to specify namespace** 🤦
   ```bash
   # ❌ Deploys to whatever namespace is current
   kubectl apply -f my-app.yaml

   # ✅ Explicit namespace
   kubectl apply -f my-app.yaml -n production
   ```

3. **Deleting a namespace in production** 😱
   > `kubectl delete namespace production` deletes EVERYTHING in it. No undo!

4. **Assuming namespaces provide security** 🤦
   > Pods in different namespaces CAN communicate by default. Use NetworkPolicies for isolation.

---

## 🎤 Interview Questions

1. **What is a Namespace in Kubernetes?**
   > A virtual cluster within a physical cluster that provides a scope for resource names and enables organization, access control, and resource quotas.

2. **What are the default Namespaces in Kubernetes?**
   > `default`, `kube-system`, `kube-public`, and `kube-node-lease`.

3. **How do you access a Service in another Namespace?**
   > Using DNS: `<service-name>.<namespace>.svc.cluster.local`

4. **Do Namespaces provide security isolation?**
   > Not by default. Pods in different namespaces can communicate unless you configure NetworkPolicies.

5. **What happens when you delete a Namespace?**
   > All resources (pods, services, deployments, etc.) inside the namespace are deleted permanently.

---

## 📝 Summary

| Concept | What to Remember |
|---------|-----------------|
| **Namespace** | Virtual cluster within a cluster |
| **Default NS** | `default`, `kube-system`, `kube-public`, `kube-node-lease` |
| **DNS format** | `<svc>.<ns>.svc.cluster.local` |
| **ResourceQuota** | Limits resources per namespace |
| **Isolation** | Not security by default — use NetworkPolicies |
| **Best practice** | Never use `default` namespace |

> *"Ravi, Namespaces are like organizing files into folders. Without them, your cluster becomes a messy desktop with 1000 icons. Keep it organized from day one! 📂"*

---

## 🔗 Navigation

| ← Previous | Home | Next → |
|:----------:|:----:|:------:|
| **[← 09: Services](../09-Services/README.md)** | [📚 Home](../README.md) | **[11: ConfigMaps →](../11-ConfigMaps/README.md)** |

> 💡 **Tip:** Open the next topic in a new tab so you can come back to this one anytime!
