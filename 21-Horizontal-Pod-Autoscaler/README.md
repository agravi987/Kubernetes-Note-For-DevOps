# 📈 Horizontal Pod Autoscaler (HPA)

> *"Ravi, traffic to your app isn't constant. At midnight it's quiet, at lunch it's busy, during a sale it goes CRAZY. You could manually scale... or you could let HPA do it automatically! It watches your metrics and scales pods up/down. Like magic! 🪄"*

---

## 🤔 What is HPA?

**Horizontal Pod Autoscaler** automatically scales the number of pod replicas based on observed metrics (CPU, memory, or custom metrics).

```
Low traffic → 2 pods  (HPA: "Enough!")
Busy day    → 20 pods (HPA: "Need more!")
Night time  → 3 pods  (HPA: "Can reduce!")
```

---

## 💡 Why Do We Need HPA?

| Without HPA | With HPA |
|------------|---------|
| Manual scaling 🤦 | Automatic scaling 🤖 |
| Over-provisioned (wasted money) 💸 | Right-sized (save money) 💰 |
| Under-provisioned (downtime) 😱 | Scales up before issues ⚡ |
| 24/7 monitoring needed 👀 | Self-managing 🎉 |

---

## 📝 YAML Example

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-deployment       # Target deployment to scale
  minReplicas: 2              # Never go below 2
  maxReplicas: 10             # Never go above 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70   # Scale when CPU > 70%
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80   # Scale when memory > 80%
```

### Quick Create (Command Line)

```bash
kubectl autoscale deployment web-deployment \
  --min=2 \
  --max=10 \
  --cpu-percent=70
```

---

## 🔄 How It Works

```
1. Metrics Server collects CPU/memory data every 15s
2. HPA controller checks metrics every 30s (default)
3. Compares current utilization with target utilization
4. If CPU > target → Scale UP
5. If CPU < target → Scale DOWN (wait 5 min cooldown)
```

### Scaling Calculation

```
Target CPU: 70%
Current CPU: 140%
Current replicas: 2

Needed replicas = ceil(2 × (140/70)) = ceil(4) = 4

HPA scales from 2 → 4 replicas
```

> 😄 **Joke:** HPA is like an Indian cricket captain — when the batting is going well (low CPU), he keeps the same team. When wickets fall (high CPU), he calls in the next batsman (pod) immediately! Kohli-level auto-scaling! 🏏💪

---

## 🧠 Important Concepts

### Metrics Server Dependency

> ⚠️ HPA requires **Metrics Server** to be installed! Without it, HPA can't see CPU/memory usage.

```bash
# Check if metrics-server is installed
kubectl top nodes
kubectl top pods

# Install on Minikube
minikube addons enable metrics-server

# Install manually
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

### Stabilization Window

```yaml
behavior:
  scaleDown:
    stabilizationWindowSeconds: 300  # Wait 5 min before scaling down
    policies:
      - type: Percent
        value: 10                    # Scale down max 10% at a time
        periodSeconds: 60
  scaleUp:
    stabilizationWindowSeconds: 0    # Scale up immediately
    policies:
      - type: Percent
        value: 100                   # Can double the replicas
        periodSeconds: 15
```

### Scaling Policies

```yaml
behavior:
  scaleUp:
    policies:
      - type: Percent
        value: 100    # Max 100% increase
      - type: Pods
        value: 4      # OR max 4 new pods
      periodSeconds: 15
  scaleDown:
    policies:
      - type: Percent
        value: 10     # Max 10% decrease
      periodSeconds: 60
```

---

## 📋 Best Practices

- ✅ **Always set minReplicas ≥ 2** — avoid single pod failure
- ✅ **Set realistic resource requests** — HPA uses them for calculations
- ✅ **Use custom metrics** for more accurate scaling (e.g., queue length)
- ✅ **Configure stabilization windows** to prevent flapping
- ✅ **Monitor HPA behavior** with `kubectl describe hpa`
- ❌ **Don't set minReplicas = 1** in production
- ❌ **Don't scale based on memory** for most apps (CPU is more responsive)
- ❌ **Don't set targets too low** — you'll scale constantly (thrashing)

---

## ❌ Common Mistakes

1. **No Metrics Server installed** 🤦
   ```bash
   kubectl top nodes
   # error: Metrics API not available

   # HPA won't work without metrics!
   ```

2. **Resource requests not set** 🤦
   > HPA calculates utilization as a percentage of requests. If requests are 0, HPA can't work!

3. **Targets too aggressive** 🤦
   > CPU target: 30% → HPA scales up for every tiny spike → waste of resources.

4. **No cooldown period** 🤦
   > Without stabilization windows, HPA can flap between scaling up and down rapidly.

---

## 🎤 Interview Questions

1. **What is HPA?**
   > Horizontal Pod Autoscaler automatically scales the number of pod replicas based on metrics like CPU and memory utilization.

2. **What is required for HPA to work?**
   > Metrics Server must be installed. The target deployment must have resource requests defined.

3. **How does HPA calculate the number of replicas?**
   > `desiredReplicas = ceil(currentReplicas × (currentMetric / targetMetric))`

4. **How do you prevent scaling flapping?**
   > Set stabilization windows and scaling policies in the HPA behavior section.

5. **Can HPA scale based on custom metrics?**
   > Yes, with custom metrics adapters (e.g., Prometheus adapter), HPA can scale on queue length, request rate, etc.

---

## 📝 Summary

| Concept | What to Remember |
|---------|-----------------|
| **HPA** | Auto-scales pods based on metrics |
| **Prerequisite** | Metrics Server + resource requests |
| **Metrics** | CPU, memory, or custom |
| **Scaling** | Based on utilization percentage |
| **Cooldown** | Stabilization windows prevent flapping |
| **Behavior** | Fine-tune scale up/down speed |

> *"Ravi, HPA is how you handle traffic spikes without losing sleep. Set it up with reasonable targets, configure stabilization windows, and let Kubernetes do the heavy lifting! 📈"*

---

## 🔗 Navigation

| ← Previous | Home | Next → |
|:----------:|:----:|:------:|
| **[← 20: Resource Management](../20-Resource-Management/README.md)** | [📚 Home](../README.md) | **[22: Scheduling →](../22-Scheduling/README.md)** |

> 💡 **Tip:** Open the next topic in a new tab so you can come back to this one anytime!
