# 📋 Scheduling

> *"Ravi, you've got a cluster with 50 nodes. You create a pod. WHERE does it go? That's the Scheduler's job! But you can also INFLUENCE where your pods land using node selectors, affinity rules, taints, and tolerations. Let's master pod placement! 📋"*

<div align="center">

| 📖 Reading Time | 🎯 Difficulty | 🏷️ Category |
|:---:|:---:|:---:|
| ~7 min | ⭐⭐⭐ Advanced | Operations |

</div>

---

## 🤔 What is Scheduling?

Scheduling is the process of **assigning pods to nodes** in the cluster based on resource requirements and constraints.

```
Pod needs: 500m CPU, 1Gi RAM
Scheduler checks:
  ├── Node 1: 200m free ❌ (not enough)
  ├── Node 2: 1000m free ✅ (enough!)
  └── Node 3: 2000m free ✅ (enough!)

Scheduler picks Node 2 (best fit, not overprovisioned)
```

---

## 💡 Why Do We Need Scheduling Control?

| Default Scheduling | With Scheduling Rules |
|-------------------|---------------------|
| Random-looking placement 🤷 | Strategic placement 🎯 |
| All pods on same node ⚠️ | Spread across nodes ⚖️ |
| Database on cheap node 😰 | Database on SSD node 💾 |
| No hardware affinity 🤦 | GPU pods on GPU nodes 🖥️ |

---

## 📚 Scheduling Tools

### 1. nodeSelector (Simple)

```yaml
# Pod goes ONLY to nodes with label disktype=ssd
spec:
  nodeSelector:
    disktype: ssd

# Label a node first:
kubectl label node node-1 disktype=ssd
```

---

### 2. Node Affinity (Advanced)

```yaml
spec:
  affinity:
    nodeAffinity:
      # MUST match (hard requirement)
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: zone
                operator: In
                values: ["us-east-1a", "us-east-1b"]

      # SHOULD match (soft preference)
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 80
          preference:
            matchExpressions:
              - key: disktype
                operator: In
                values: ["ssd"]
```

| Operator | Description |
|----------|-------------|
| `In` | Value is in the list |
| `NotIn` | Value is NOT in the list |
| `Exists` | Label exists |
| `DoesNotExist` | Label doesn't exist |
| `Gt` | Greater than |
| `Lt` | Less than |

---

### 3. Pod Affinity / Anti-Affinity

```yaml
# Pod Affinity: co-locate with specific pods
affinity:
  podAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
            - key: app
              operator: In
              values: ["cache"]
        topologyKey: kubernetes.io/hostname

# Pod Anti-Affinity: NEVER co-locate with specific pods
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
              - key: app
                operator: In
                values: ["web"]
           topologyKey: kubernetes.io/hostname
```

> 💡 **Tip:** Use anti-affinity to spread replicas across nodes for high availability!

> 😄 **Joke:** Pod Anti-Affinity is like the Indian wedding seating planner — you SPECIFICALLY make sure the two aunties who always argue are on opposite sides of the hall. No drama on the same node! 💒🪑

---

### 4. Taints and Tolerations

**Taints** repel pods from nodes. **Tolerations** allow pods to tolerate taints.

```bash
# Add a taint to a node
kubectl taint nodes node-1 dedicated=gpu:NoSchedule
```

```yaml
# Pod tolerates the taint
spec:
  tolerations:
    - key: "dedicated"
      operator: "Equal"
      value: "gpu"
      effect: "NoSchedule"
```

| Taint Effect | Behavior |
|-------------|----------|
| **NoSchedule** | New pods won't be scheduled (existing pods unaffected) |
| **PreferNoSchedule** | K8s will try to avoid scheduling (soft) |
| **NoExecute** | New pods won't schedule AND existing pods are evicted |

---

## 📋 Best Practices

- ✅ **Use nodeSelector** for simple cases (one rule)
- ✅ **Use nodeAffinity** for complex cases (multiple rules)
- ✅ **Use podAntiAffinity** to spread replicas across nodes
- ✅ **Use taints** to reserve nodes for specific workloads
- ✅ **Label nodes** for easy selection (environment, hardware, zone)
- ❌ **Don't over-specify constraints** — pod may never be scheduled
- ❌ **Don't use node affinity when not needed** — adds complexity

---

## ❌ Common Mistakes

1. **Over-constraining pods** 🤦
   > Too many affinity rules → no node matches → pod stays Pending forever.

2. **Forgetting to label nodes** 🤦
   > nodeSelector needs labels! `kubectl label node node-1 disktype=ssd`

3. **Using NoExecute taint carelessly** 🔥
   > NoExecute evicts EXISTING pods too! Can cause downtime.

4. **Not checking pod scheduling** 🤦
   ```bash
   kubectl describe pod <pod-name>   # Check Events for scheduling issues
   kubectl get events | grep FailedScheduling
   ```

---

## 🎤 Interview Questions

1. **What is the difference between nodeSelector and nodeAffinity?**
   > nodeSelector is simple key-value matching. nodeAffinity supports complex expressions (In, NotIn, Exists, Gt, etc.) and has required/preferred modes.

2. **What is the difference between Pod Affinity and Anti-Affinity?**
   > Affinity co-locates pods together. Anti-Affinity spreads pods apart.

3. **What are Taints and Tolerations?**
   > Taints repel pods from nodes. Tolerations allow specific pods to bypass taints.

4. **What does the NoSchedule taint effect do?**
   > Prevents new pods from being scheduled on the tainted node. Existing pods are not affected.

---

## 📝 Summary

| Tool | Complexity | Use Case |
|------|-----------|----------|
| **nodeSelector** | Simple | Basic node selection |
| **nodeAffinity** | Advanced | Complex node rules |
| **podAffinity** | Advanced | Co-locate related pods |
| **podAntiAffinity** | Advanced | Spread pods for HA |
| **Taints** | Intermediate | Reserve nodes |

> *"Ravi, for most apps, nodeSelector is enough. Use podAntiAffinity to spread replicas. Only dive into advanced affinity when you have specific requirements. Keep it simple! 📋"*

---

## 🔗 Navigation

| ← Previous | Home | Next → |
|:----------:|:----:|:------:|
| **[← 21: HPA](../21-Horizontal-Pod-Autoscaler/README.md)** | [📚 Home](../README.md) | **[23: RBAC →](../23-RBAC/README.md)** |

> 💡 **Tip:** Open the next topic in a new tab so you can come back to this one anytime!
