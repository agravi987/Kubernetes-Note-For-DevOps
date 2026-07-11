# 🏷️ Labels and Selectors

> *"Hey Ravi, imagine you have 1000 boxes in a warehouse. Without labels, finding the right box is a nightmare. Labels and Selectors are Kubernetes' way of organizing and finding resources. Simple concept, MASSIVE impact!"*

---

## 🤔 What are Labels?

Labels are **key-value pairs** attached to Kubernetes objects (pods, services, deployments, etc.). They're like **tags or stickers** you put on resources to identify and organize them.

```yaml
metadata:
  labels:
    app: web
    env: production
    tier: frontend
```

- **Key**: `app`, `env`, `tier` (you choose these)
- **Value**: `web`, `production`, `frontend` (you choose these)
- An object can have **many** labels

---

## 💡 Why Do We Need Labels?

Imagine you have 50 pods running different apps:

```
Without Labels:
pod-1, pod-2, pod-3, pod-4, pod-5... pod-50

🤔 Which pod runs the frontend?
🤔 Which pods are in production?
🤔 Which pods need to be updated?
```

```
With Labels:
pod-1 (app=web, env=prod)
pod-2 (app=api, env=prod)
pod-3 (app=web, env=staging)
pod-4 (app=db, env=prod)

✅ Easy! Show me all pods where app=web
✅ Easy! Update all pods where env=staging
```

**Labels power everything in Kubernetes:**
- Services find pods via label selectors
- Deployments manage pods via label selectors
- Scheduling uses labels for node affinity
- Network Policies use labels to define traffic rules

---

## 🔄 How Labels Work

```
┌─────────────────────────────────────────┐
│              Object                     │
│  ┌───────────────────────────────────┐  │
│  │ metadata:                         │  │
│  │   name: my-pod                    │  │
│  │   labels:                         │  │
│  │     app: web          ◄─────────────── Used by Selectors
│  │     env: production   ◄─────────────── to find this pod
│  │     version: v2       ◄───────────────
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
                    │
                    ▼
        ┌───────────────────────┐
        │      Selectors       │
        │  app = web           │──→ Finds pod-1, pod-3
        │  env = production    │──→ Finds pod-1, pod-2, pod-4
        └───────────────────────┘
```

---

## 📚 Labels

### Label Rules

```yaml
# ✅ Valid labels
app: web
app-tier: frontend
app.kubernetes.io/name: web
app.kubernetes.io/version: "1.0"
example.com/team: backend

# ❌ Invalid labels
# Key must be <= 63 characters
# Key must start with alphanumeric
# Key can only contain: alphanumeric, -, _, .
# Value must be <= 63 characters
# Value can be empty
```

### Standard Kubernetes Label Conventions

```yaml
metadata:
  labels:
    # Recommended standard labels
    app.kubernetes.io/name: web
    app.kubernetes.io/instance: web-abc123
    app.kubernetes.io/version: "1.0"
    app.kubernetes.io/component: frontend
    app.kubernetes.io/part-of: my-platform
    app.kubernetes.io/managed-by: kubectl
```

> 💡 **Ravi, you don't HAVE to use these conventions**, but they're useful in larger teams for consistency.

---

## 🔍 Selectors

Selectors are **filter expressions** used to find objects by their labels.

### Equality-Based Selectors

```bash
# Find pods with label app=web
kubectl get pods -l app=web

# Find pods with label env != production
kubectl get pods -l env!=production

# Find pods with label tier (exists, any value)
kubectl get pods -l tier

# Find pods WITHOUT label tier
kubectl get pods -l '!tier'
```

### Set-Based Selectors

```bash
# Find pods where app is either web or api
kubectl get pods -l 'app in (web, api)'

# Find pods where env is NOT staging
kubectl get pods -l 'env notin (staging)'

# Multiple conditions (AND logic)
kubectl get pods -l 'app=web,env=production'
```

### Selector Summary

| Operator | Description | Example |
|----------|-------------|---------|
| `=` or `==` | Equal to | `app=web` |
| `!=` | Not equal to | `env!=staging` |
| `in` | In a set | `app in (web, api)` |
| `notin` | Not in a set | `env notin (dev, test)` |
| `exists` | Label exists | `tier` |
| `!` | Label doesn't exist | `!debug` |

---

## 📝 YAML Examples

### Labels on a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
  labels:
    app: web
    env: production
    tier: frontend
spec:
  containers:
    - name: nginx
      image: nginx:1.25
```

### Labels on a Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
  labels:
    app: web
    managed-by: team-frontend
spec:
  replicas: 3
  selector:
    matchLabels:        # ← This Deployment FINDS pods with these labels
      app: web          #    (must match the pod template labels)
  template:
    metadata:
      labels:           # ← These labels are APPLIED to each pod
        app: web
        env: production
    spec:
      containers:
        - name: nginx
          image: nginx:1.25
```

> ⚠️ **Critical Rule, Ravi:** `spec.selector.matchLabels` MUST match `spec.template.metadata.labels`. If they don't match, Kubernetes will reject the deployment!

### Service Selecting Pods

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:           # ← Service finds pods with these labels
    app: web          #    All traffic to service → goes to matching pods
  ports:
    - port: 80
      targetPort: 80
```

---

## 🧠 Important Concepts

### Labels ≠ Annotations

| Feature | Labels | Annotations |
|---------|--------|-------------|
| **Purpose** | Identify & select objects | Store metadata for tools |
| **Used by** | Kubernetes (selectors) | External tools, humans |
| **Indexed** | Yes (fast lookup) | No (not for selection) |
| **Example** | `app: web` | `description: "Main web app"` |

```yaml
metadata:
  labels:
    app: web              # ← Used by Kubernetes to find this pod
  annotations:
    description: "This is the main web application"
    contact: "ravi@company.com"
    prometheus.io/scrape: "true"
```

> 😄 **Joke:** Labels are like Indian wedding invitations — you put every relation on there (cousin's wife's brother's friend), but at the end of the day, only the people with the RIGHT label get through the door! 🎉📋

### Label Selectors in Different Resources

| Resource | How Selectors Are Used |
|----------|----------------------|
| **Deployment** | `matchLabels` finds pods to manage |
| **Service** | `selector` finds pods to send traffic to |
| **NetworkPolicy** | `podSelector` finds pods to apply rules to |
| **HPA** | Targets a deployment (by name) |

---

## 📋 Best Practices

- ✅ **Use consistent label naming** across your team
- ✅ **Always include `app` label** — it's the most commonly used
- ✅ **Use standard labels** like `app.kubernetes.io/name`
- ✅ **Keep labels simple** — short, meaningful values
- ✅ **Use selectors to filter** resources instead of listing everything
- ❌ **Don't use labels as the only organizational tool** — use Namespaces too
- ❌ **Don't change labels on running resources** — it can break selectors
- ❌ **Don't put sensitive data in labels** — they're visible to everyone

---

## ❌ Common Mistakes

1. **Selector mismatch in Deployments** 🔥
   ```yaml
   # ❌ WRONG — selector doesn't match template labels
   spec:
     selector:
       matchLabels:
         app: web        # Looking for app=web
     template:
       metadata:
         labels:
           app: frontend  # But pods have app=frontend!

   # ✅ CORRECT — they match
   spec:
     selector:
       matchLabels:
         app: web
     template:
       metadata:
         labels:
           app: web       # Both say app=web ✓
   ```

2. **Changing labels on live pods** 🔥
   > If a Deployment manages pods with `app=web` and you change a pod's label, the Deployment will create a NEW pod to replace it!

3. **Using too many labels** 🤷
   > Keep it simple. 3-5 labels per object is usually enough.

4. **Forgetting to filter by namespace** 🤦
   ```bash
   # This only shows pods in current namespace
   kubectl get pods -l app=web

   # This shows across ALL namespaces
   kubectl get pods -l app=web -A
   ```

---

## 🎤 Interview Questions

1. **What are Kubernetes Labels?**
   > Key-value pairs attached to objects for identification and organization. Used by selectors to find and filter resources.

2. **What is the difference between Labels and Annotations?**
   > Labels are used by Kubernetes to identify and select objects (indexed for fast lookup). Annotations store non-identifying metadata for external tools and are not used for selection.

3. **What are Label Selectors?**
   > Filter expressions used to find objects by their labels. Support equality-based (`=`, `!=`) and set-based (`in`, `notin`, `exists`) operators.

4. **Why must Deployment selector match Pod template labels?**
   > The Deployment uses selectors to find which pods it manages. If they don't match, it can't track its pods, leading to infinite pod creation or orphaned pods.

5. **How would you find all pods with label app=web in namespace production?**
   > `kubectl get pods -l app=web -n production`

---

## 📝 Summary

| Concept | What to Remember |
|---------|-----------------|
| **Labels** | Key-value pairs on objects (`app: web`) |
| **Selectors** | Filter expressions to find objects by labels |
| **Equality-based** | `app=web`, `env!=staging` |
| **Set-based** | `app in (web, api)`, `env notin (dev)` |
| **Labels vs Annotations** | Labels are for selection, Annotations are for metadata |
| **Golden Rule** | Deployment selector MUST match Pod template labels |

> *"Ravi, Labels and Selectors are deceptively simple but incredibly powerful. Every Kubernetes resource uses them. Master this concept and everything else becomes easier! 💪"*
