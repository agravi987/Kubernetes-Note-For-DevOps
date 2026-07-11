# 🔒 Network Policies

> *"Ravi, by default in Kubernetes, EVERY pod can talk to EVERY other pod. In a small dev cluster, fine. In production with sensitive databases? TERRIFYING! Network Policies are like firewalls for your pods — they control who can talk to whom. Essential for security! 🔒"*

---

## 🤔 What are Network Policies?

A **NetworkPolicy** defines rules for **which pods can communicate** with each other and with external endpoints. Think of it as a firewall for pod-to-pod traffic.

```
Without Network Policies:
  All pods ←→ All pods (everything is allowed! 😱)

With Network Policies:
  Frontend ←→ Backend ←→ Database (only allowed paths! ✅)
  Frontend ←/→ Database (blocked!)
```

---

## 💡 Why Do We Need Network Policies?

| Without Network Policies | With Network Policies |
|------------------------|---------------------|
| Any pod can access any pod 🔓 | Only allowed traffic flows 🔒 |
| Compromised pod = full access 🤦 | Blast radius is limited 🛡️ |
| No segmentation 😰 | Network segmentation ✅ |
| Compliance issues 📋 | Audit-friendly rules 🎯 |

---

## 📝 YAML Example

### Default Deny All

```yaml
# Deny ALL traffic in a namespace (start here!)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production
spec:
  podSelector: {}              # Applies to ALL pods in namespace
  policyTypes:
    - Ingress                   # Deny incoming traffic
    - Egress                    # Deny outgoing traffic
```

### Allow Specific Traffic

```yaml
# Allow frontend to talk to backend
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: backend              # Apply to pods with app=backend
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend     # Only allow from pods with app=frontend
      ports:
        - port: 8080
          protocol: TCP
```

### Complex Policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
    - Ingress
  ingress:
    # Allow from backend pods on port 3306
    - from:
        - podSelector:
            matchLabels:
              app: backend
      ports:
        - port: 3306
          protocol: TCP

    # Allow from monitoring namespace
    - from:
        - namespaceSelector:
            matchLabels:
              name: monitoring
      ports:
        - port: 9104
          protocol: TCP
```

> 😄 **Joke:** Without Network Policies, your pods are like a Indian train general compartment — everyone talks to everyone, bags are everywhere, random people share your biryani, and nobody knows what's happening. Time for some rules! 🚂😅

---

## 🧠 Important Concepts

### How Network Policies Work

```
1. First, apply a "default deny" policy to the namespace
2. Then, ADD specific allow rules
3. Network Policies are ADDITIVE (not replacement)

Multiple Network Policies:
  Policy A: Allow frontend → backend
  Policy B: Allow monitoring → all pods
  Result: BOTH rules apply (additive)
```

### Policy Types

| Type | Controls |
|------|----------|
| **Ingress** | Incoming traffic TO the pod |
| **Egress** | Outgoing traffic FROM the pod |

### Selector Types

```yaml
ingress:
  - from:
      # Pod selector — specific pods
      - podSelector:
          matchLabels:
            app: frontend

      # Namespace selector — pods in a namespace
      - namespaceSelector:
          matchLabels:
            name: monitoring

      # Combined — pods in a specific namespace with specific labels
      - podSelector:
          matchLabels:
            role: db-admin
        namespaceSelector:
          matchLabels:
            name: production

      # IP block — external IPs
      - ipBlock:
          cidr: 10.0.0.0/8
          except:
            - 10.0.1.0/24
```

---

## ⚠️ Prerequisites

> ⚠️ **Network Policies require a CNI plugin that supports them!**

| CNI Plugin | Supports NetworkPolicy? |
|-----------|----------------------|
| Calico | ✅ Yes |
| Cilium | ✅ Yes |
| Weave Net | ✅ Yes |
| Flannel | ❌ No (by default) |
| kube-proxy only | ❌ No |

```bash
# Minikube with Calico
minikube start --network-plugin=cni --cni=calico

# kind with Calico
kind create cluster --config - <<EOF
kind: Cluster
networking:
  disableDefaultCNI: true
  podSubnet: 192.168.0.0/16
EOF
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

---

## 📋 Best Practices

- ✅ **Start with default-deny** in production namespaces
- ✅ **Add specific allow rules** after denying everything
- ✅ **Use namespaces** for network segmentation
- ✅ **Test policies** before applying in production
- ✅ **Document your policies** — they're your network firewall rules
- ❌ **Don't rely on Network Policies alone** — combine with RBAC
- ❌ **Don't use Flannel** if you need Network Policies

---

## ❌ Common Mistakes

1. **Forgetting that policies are ADDITIVE** 🤦
   > Multiple policies don't replace each other — they ADD rules together.

2. **Using Flannel without knowing it doesn't support NetworkPolicy** 🤦
   > Your policies will be silently ignored!

3. **Denying all egress and breaking DNS** 🔥
   ```yaml
   # This breaks DNS resolution!
   egress: []

   # Always allow DNS (port 53)
   egress:
     - to: []
       ports:
         - port: 53
           protocol: UDP
         - port: 53
           protocol: TCP
   ```

4. **Not having a default-deny policy** 🤦
   > Without it, all traffic is allowed. Policies only ADD restrictions.

---

## 🎤 Interview Questions

1. **What is a Network Policy?**
   > A Kubernetes resource that controls pod-to-pod traffic flow, acting as a firewall for network communication.

2. **What is the default behavior without Network Policies?**
   > All pods can communicate with all other pods (no restrictions).

3. **What is the recommended approach for Network Policies?**
   > Start with a default-deny policy, then add specific allow rules as needed.

4. **What CNI plugins support Network Policies?**
   > Calico, Cilium, Weave Net. Flannel does not support them by default.

5. **What happens if you deny all egress without allowing DNS?**
   > Pods can't resolve DNS names, breaking most applications. Always allow port 53.

---

## 📝 Summary

| Concept | What to Remember |
|---------|-----------------|
| **Network Policy** | Firewall rules for pod traffic |
| **Default** | All traffic allowed |
| **Approach** | Default deny → Add allow rules |
| **Additive** | Multiple policies add up |
| **CNI Requirement** | Need Calico/Cilium (not Flannel) |
| **Always allow** | DNS (port 53) in egress rules |

> *"Ravi, Network Policies are your cluster's firewall. Start with default-deny, add specific allows, and always permit DNS. This is non-negotiable in production! 🔒"*

---

## 🔗 Navigation

| ← Previous | Home | Next → |
|:----------:|:----:|:------:|
| **[← 23: RBAC](../23-RBAC/README.md)** | [📚 Home](../README.md) | **[25: Debugging →](../25-Debugging/README.md)** |

> 💡 **Tip:** Open the next topic in a new tab so you can come back to this one anytime!
