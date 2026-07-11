# 🔑 RBAC (Role-Based Access Control)

> *"Ravi, you wouldn't give every employee the same key card at your office, right? The intern doesn't need access to the CEO's office. RBAC in Kubernetes works the same way — it controls WHO can do WHAT on WHICH resources. Security is not optional! 🔐"*

<div align="center">

| 📖 Reading Time | 🎯 Difficulty | 🏷️ Category |
|:---:|:---:|:---:|
| ~8 min | ⭐⭐⭐ Advanced | Security |

</div>

---

## 🤔 What is RBAC?

**Role-Based Access Control** defines who (Subject) can do what (Role) on which resources (Bindings).

```
RBAC = Subject + Role + Binding

Subject:  WHO?        → User, Group, ServiceAccount
Role:     WHAT?       → Get, List, Create, Delete, etc.
Binding:  CONNECTION  → Links Subject to Role
```

---

## 💡 Why Do We Need RBAC?

| Without RBAC | With RBAC |
|-------------|----------|
| Everyone can do everything 😱 | Least-privilege access 🔒 |
| Accidental deletion possible 🤦 | Only authorized actions allowed ✅ |
| No audit trail 🤷 | Clear permissions tracking 📋 |
| One compromised account = game over 🔥 | Damage is limited 🛡️ |

---

## 📚 RBAC Components

### 1. Roles (Namespace-scoped)

```yaml
# Role: permissions within a SINGLE namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: development
  name: pod-reader
rules:
  - apiGroups: [""]             # "" = core API group
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["pods/log"]
    verbs: ["get"]
```

### 2. ClusterRoles (Cluster-scoped)

```yaml
# ClusterRole: permissions across ALL namespaces + cluster resources
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["namespaces"]
    verbs: ["get", "list"]
```

### 3. RoleBinding

```yaml
# Bind a Role to a User/Group/ServiceAccount
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: development
subjects:
  - kind: User
    name: ravi
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### 4. ClusterRoleBinding

```yaml
# Bind a ClusterRole across the cluster
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-all-nodes
subjects:
  - kind: User
    name: ravi
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

---

## 🧠 Important Concepts

### API Groups & Resources

```yaml
rules:
  - apiGroups: [""]                    # Core group (pods, services, etc.)
    resources: ["pods"]
    verbs: ["get", "list"]

  - apiGroups: ["apps"]                # Apps group (deployments, etc.)
    resources: ["deployments"]
    verbs: ["get", "list", "create"]

  - apiGroups: ["networking.k8s.io"]   # Network group
    resources: ["ingresses"]
    verbs: ["get", "list"]
```

### Verbs

| Verb | Description |
|------|-------------|
| `get` | Read a specific resource |
| `list` | List resources |
| `watch` | Watch for changes |
| `create` | Create new resources |
| `update` | Update existing resources |
| `patch` | Partial update |
| `delete` | Delete a resource |
| `deletecollection` | Delete multiple resources |

### ServiceAccounts

```yaml
# Create a ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-service-account
  namespace: development

---
# Bind RBAC to the ServiceAccount
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-access
  namespace: development
subjects:
  - kind: ServiceAccount
    name: app-service-account
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

> 😄 **Joke:** Giving everyone cluster-admin access is like giving your entire pg (paying guest) the master key to the kitchen fridge — sure, trust is great, but someone WILL eat your biryani at 2 AM. Guard those permissions! 🍛🔐

---

## 📚 kubectl Commands

```bash
# Check what you can do
kubectl auth can-i create pods
kubectl auth can-i delete deployments --namespace=production

# Check all permissions for current user
kubectl auth can-i --list

# Check who can do what
kubectl auth can-i --list --namespace=development

# Create RoleBinding from command line
kubectl create rolebinding pod-reader \
  --user=ravi \
  --role=pod-reader \
  --namespace=development
```

---

## 📋 Best Practices

- ✅ **Follow least-privilege principle** — give minimum required permissions
- ✅ **Use Roles** for namespace-scoped access (not ClusterRoles)
- ✅ **Use ServiceAccounts** for applications (not default SA)
- ✅ **Audit RBAC regularly** — check who has access to what
- ✅ **Use Groups** instead of individual Users when possible
- ❌ **Don't give cluster-admin to everyone** 🚫
- ❌ **Don't use the default ServiceAccount** for applications
- ❌ **Don't create overly broad Roles** (`*` on all resources)

---

## ❌ Common Mistakes

1. **Using cluster-admin too broadly** 🔥
   > cluster-admin has FULL access. Only give it to cluster operators!

2. **Forgetting that RBAC doesn't restrict existing connections** 🤦
   > If a user already has a connection, changing RBAC doesn't immediately kill it.

3. **Confusing Role vs ClusterRole** 🤦
   > Role = single namespace. ClusterRole = all namespaces + cluster resources.

4. **Using default ServiceAccount** 🤦
   > Create dedicated ServiceAccounts for each application with specific permissions.

---

## 🎤 Interview Questions

1. **What is RBAC?**
   > Role-Based Access Control — defines who can perform what actions on which resources.

2. **What is the difference between Role and ClusterRole?**
   > Role is namespace-scoped. ClusterRole is cluster-scoped (all namespaces + cluster resources).

3. **What is the difference between RoleBinding and ClusterRoleBinding?**
   > RoleBinding links a Role to subjects in a namespace. ClusterRoleBinding links a ClusterRole to subjects cluster-wide.

4. **What is a ServiceAccount?**
   > An identity for processes/pods running in the cluster. Different from User accounts (for humans).

5. **What is the least-privilege principle?**
   > Giving only the minimum permissions needed to perform a task. Nothing more.

---

## 📝 Summary

| Component | Scope | Purpose |
|-----------|-------|---------|
| **Role** | Namespace | Define permissions |
| **ClusterRole** | Cluster | Define cluster-wide permissions |
| **RoleBinding** | Namespace | Link Role to subjects |
| **ClusterRoleBinding** | Cluster | Link ClusterRole to subjects |
| **ServiceAccount** | Namespace | Identity for pods |

> *"Ravi, RBAC is your first line of defense in Kubernetes. Start with no access, add permissions as needed. NEVER start with full access and try to remove it later. Security first! 🔐"*

---

## 🔗 Navigation

| ← Previous | Home | Next → |
|:----------:|:----:|:------:|
| **[← 22: Scheduling](../22-Scheduling/README.md)** | [📚 Home](../README.md) | **[24: Network Policies →](../24-Network-Policies/README.md)** |

> 💡 **Tip:** Open the next topic in a new tab so you can come back to this one anytime!
