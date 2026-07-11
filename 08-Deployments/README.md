# 🚀 Deployments

> *"Ravi, THIS is the resource you'll use 90% of the time! Deployments are the golden standard for running applications in Kubernetes. They manage ReplicaSets, handle rolling updates, support rollbacks, and make your life easy. Let's master it!"*

---

## 🤔 What is a Deployment?

A **Deployment** provides declarative updates for Pods and ReplicaSets. It's the most common way to run applications in Kubernetes.

```
Deployment
├── Creates and manages ReplicaSets
├── Rolls out updates gradually (rolling update)
├── Supports instant rollbacks
├── Scales up/down easily
└── Pauses/resumes deployments
```

---

## 💡 Why Do We Need Deployments?

| Without Deployment | With Deployment |
|-------------------|----------------|
| Update app → downtime 😰 | Rolling update → zero downtime 🎉 |
| Bug in new version → manual rollback | One command rollback 🔄 |
| Scale manually | Declarative scaling 📈 |
| Track changes? Good luck! | Revision history built-in 📋 |

---

## 🔄 How It Works

### Rolling Update (Default Strategy)

```
Step 1: Start with 3 pods running v1
┌─────┐  ┌─────┐  ┌─────┐
│ v1  │  │ v1  │  │ v1  │
└─────┘  └─────┘  └─────┘

Step 2: Create 1 pod with v2 (maxSurge=1)
┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐
│ v1  │  │ v1  │  │ v1  │  │ v2  │
└─────┘  └─────┘  └─────┘  └─────┘

Step 3: Delete 1 v1 pod (maxUnavailable=0)
┌─────┐  ┌─────┐  ┌─────┐
│ v1  │  │ v1  │  │ v2  │
└─────┘  └─────┘  └─────┘

Step 4: Create another v2 pod
┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐
│ v1  │  │ v1  │  │ v2  │  │ v2  │
└─────┘  └─────┘  └─────┘  └─────┘

Step 5: Delete another v1 pod
┌─────┐  ┌─────┐  ┌─────┐
│ v1  │  │ v2  │  │ v2  │
└─────┘  └─────┘  └─────┘

Step 6: Continue until all pods are v2
┌─────┐  ┌─────┐  ┌─────┐
│ v2  │  │ v2  │  │ v2  │
└─────┘  └─────┘  └─────┘

✅ Zero downtime! At least 3 pods always running!
```

---

## 📝 YAML Example

### Basic Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
  labels:
    app: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
        env: production
    spec:
      containers:
        - name: nginx
          image: nginx:1.25
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "250m"
              memory: "256Mi"
```

### Deployment with Rolling Update Strategy

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Max pods ABOVE desired count during update
      maxUnavailable: 0  # Max pods BELOW desired count during update
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

## 🧠 Important Concepts

### Rolling Update Parameters

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1        # Create 1 extra pod during update
    maxUnavailable: 0  # Never have fewer pods than desired
```

```
maxSurge & maxUnavailable Explained:

replicas: 3, maxSurge: 1, maxUnavailable: 0

During update:
  Maximum pods at any time: 3 + 1 = 4 (maxSurge)
  Minimum pods at any time: 3 - 0 = 3 (maxUnavailable)
  → Always at least 3 pods running! ✅
```

> 🔥 **Fun Fact / Joke:** Using :latest tag in production is like ordering 'surprise biryani' from a roadside stall — sometimes you get chicken, sometimes you get... something else entirely. Always order (pin) what you know! 🍚😂

### Recreate Strategy (All at Once)

```yaml
strategy:
  type: Recreate
  # Deletes ALL old pods BEFORE creating new ones
  # ⚠️ Causes DOWNTIME! Use only when necessary.
```

### Revision History

```bash
# See all revisions
kubectl rollout history deployment/web-deployment

# REVISION   CHANGE-CAUSE
# 1          <none>
# 2          <none>
# 3          <none>
```

---

## 📚 kubectl Commands

### Deployment Management

```bash
# Create a deployment
kubectl create deployment web --image=nginx:1.25 --replicas=3

# List deployments
kubectl get deployments

# Describe a deployment
kubectl describe deployment web

# Delete a deployment
kubectl delete deployment web

# Apply from YAML
kubectl apply -f deployment.yaml
```

### Updates & Rollouts

```bash
# Update image (triggers rolling update)
kubectl set image deployment/web nginx=nginx:1.26

# Watch rollout progress
kubectl rollout status deployment/web

# See rollout history
kubectl rollout history deployment/web

# See details of a specific revision
kubectl rollout history deployment/web --revision=2

# Pause a rollout (useful for multiple changes)
kubectl rollout pause deployment/web

# Resume a paused rollout
kubectl rollout resume deployment/web
```

### Rollbacks

```bash
# Rollback to previous version
kubectl rollout undo deployment/web

# Rollback to a specific revision
kubectl rollout undo deployment/web --to-revision=2
```

### Scaling

```bash
# Scale via kubectl
kubectl scale deployment web --replicas=5

# Autoscale (covered in HPA chapter)
kubectl autoscale deployment web --min=3 --max=10 --cpu-percent=80
```

---

## 📋 Best Practices

- ✅ **Always use Deployments** — never create bare Pods or ReplicaSets
- ✅ **Set resource requests and limits** on every container
- ✅ **Use readiness probes** so traffic only goes to ready pods
- ✅ **Use `maxSurge: 1, maxUnavailable: 0`** for zero-downtime updates
- ✅ **Use labels consistently** across deployments and services
- ✅ **Tag images with specific versions** — never use `latest` in production
- ❌ **Don't use `:latest` tag** in production (it's unpredictable)
- ❌ **Don't skip rollback testing** — always know how to rollback
- ❌ **Don't set replicas too low** — at least 2 for production (3 is better)

---

## ❌ Common Mistakes

1. **Using `:latest` tag** 🔥
   ```yaml
   # ❌ Dangerous — which version is "latest"?
   image: nginx:latest

   # ✅ Always pin versions
   image: nginx:1.25.3
   ```

2. **Selector mismatch** 🔥
   ```yaml
   # ❌ This will FAIL
   spec:
     selector:
       matchLabels:
         app: web
     template:
       metadata:
         labels:
           app: frontend  # Mismatch!

   # ✅ They must match
   spec:
     selector:
       matchLabels:
         app: web
     template:
       metadata:
         labels:
           app: web      # Same!
   ```

3. **No resource limits** 💸
   > One pod can eat all node resources. ALWAYS set requests and limits.

4. **Not monitoring rollouts** 🤦
   ```bash
   # Always watch your rollout!
   kubectl rollout status deployment/web -w
   ```

5. **Forgetting how to rollback** 😱
   ```bash
   # Remember this command:
   kubectl rollout undo deployment/web
   # It can save your production!
   ```

---

## 🎤 Interview Questions

1. **What is a Deployment?**
   > A Kubernetes object that provides declarative updates for Pods and ReplicaSets. It manages rolling updates, rollbacks, and scaling.

2. **What is the default update strategy for Deployments?**
   > RollingUpdate — gradually replaces old pods with new ones to ensure zero downtime.

3. **What is the difference between maxSurge and maxUnavailable?**
   > `maxSurge` is the maximum number of extra pods created during update. `maxUnavailable` is the maximum number of pods that can be unavailable (below desired count).

4. **How do you perform a rollback?**
   > `kubectl rollout undo deployment/<name>` — rolls back to the previous revision.

5. **What is the difference between RollingUpdate and Recreate strategy?**
   > RollingUpdate gradually replaces pods (zero downtime). Recreate deletes all old pods first, then creates new ones (causes downtime).

6. **How do you update a Deployment's container image?**
   > `kubectl set image deployment/<name> <container>=<new-image>` or edit the YAML and apply.

7. **What happens when you apply a Deployment YAML with no changes?**
   > Nothing — it's idempotent. K8s detects no changes and does nothing.

---

## 📝 Summary

| Concept | What to Remember |
|---------|-----------------|
| **Deployment** | Manages ReplicaSets, pods, updates, rollbacks |
| **RollingUpdate** | Default strategy, zero downtime |
| **Recreate** | Destroys all pods first (downtime) |
| **maxSurge** | Extra pods during update |
| **maxUnavailable** | Pods below desired count |
| **Rollback** | `kubectl rollout undo` |
| **Scaling** | `kubectl scale` or edit YAML |
| **Rule #1** | Always use Deployments, never bare Pods |

> *"Ravi, Deployments are your bread and butter in Kubernetes. You'll create hundreds of them. Master the rollout commands, always set resource limits, and NEVER use `:latest` in production. You're doing great! 🎉"*

---

## 🔗 Navigation

| ← Previous | Home | Next → |
|:----------:|:----:|:------:|
| **[← 07: ReplicaSets](../07-ReplicaSets/README.md)** | [📚 Home](../README.md) | **[Next: 09: Services →](../09-Services/README.md)** |

> 💡 **Tip:** Open the next topic in a new tab so you can come back to this one anytime!
