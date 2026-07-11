# 🔍 Debugging

> *"Ravi, things WILL go wrong in Kubernetes. Pods crash, services don't respond, deployments get stuck. Knowing how to debug quickly is what separates a junior from a senior. Let's build your debugging toolkit! 🔍"*

---

## 🤔 What is it?

Debugging in Kubernetes means **systematically investigating** why resources aren't working as expected — from pod crashes to network issues to configuration errors.

---

## 💡 Why Do We Need It?

- Pods crash at 3 AM 😰
- Deployments get stuck in progress 🤦
- Services can't reach pods 🤷
- Applications work locally but fail in K8s 😱

**Debugging skills = less downtime + happier users**

---

## 🔧 Debugging Toolkit

### Step 1: Check the Big Picture

```bash
# Are nodes healthy?
kubectl get nodes
kubectl describe node <node-name>

# Are system pods running?
kubectl get pods -n kube-system

# Any recent events?
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get events -w    # Watch in real-time
```

### Step 2: Check the Resource

```bash
# Pod status
kubectl get pods
kubectl get pods -o wide    # Shows node, IP

# Deployment status
kubectl get deployments

# Service endpoints
kubectl get endpoints my-service

# Are endpoints empty? → Selector mismatch!
```

### Step 3: Describe (The Most Important Command!)

```bash
kubectl describe pod <pod-name>
```

```
Focus on these sections:
├── State        → Running, Waiting, Terminated
├── Ready        → True or False
├── Restart Count → High count = problem!
├── Last State   → Previous crash info
└── Events       → What Kubernetes did (golden section! 🏆)
```

### Step 4: Check Logs

```bash
# Current logs
kubectl logs <pod-name>

# Previous container (after crash)
kubectl logs <pod-name> --previous

# Follow logs
kubectl logs -f <pod-name>

# Logs from specific container (multi-container pod)
kubectl logs <pod-name> -c <container-name>

# Logs from all pods matching a label
kubectl logs -l app=web --all-containers
```

### Step 5: Shell Into the Pod

```bash
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec -it <pod-name> -- sh   # For Alpine containers

# Test connectivity from inside the pod
curl http://my-service:8080
nslookup my-service
ping 10.96.0.1
```

> 😄 **Joke:** Debugging Kubernetes without checking Events is like trying to find your phone in the dark without turning on the flashlight — you'll bump into everything, blame everyone, and it was under the cushion the whole time. CHECK EVENTS! 🔦📱

---

## 🧠 Common Issues & Solutions

### 1. Pod Stuck in Pending

```bash
kubectl describe pod <pod-name> | grep -A5 "Events"
```

| Cause | Solution |
|-------|---------|
| Not enough resources | Scale up nodes or reduce pod requests |
| Node selector mismatch | Check node labels |
| Taints without tolerations | Add tolerations or remove taints |
| PVC not bound | Check PV/PVC status |

### 2. Pod in CrashLoopBackOff

```bash
# Check why it's crashing
kubectl logs <pod-name> --previous
kubectl describe pod <pod-name>
```

| Cause | Solution |
|-------|---------|
| App error/crash | Fix the application code |
| OOMKilled | Increase memory limit |
| Missing ConfigMap/Secret | Create the resource |
| Wrong command/args | Fix container command |
| Port conflict | Check containerPort |

### 3. Pod in ImagePullBackOff

```bash
kubectl describe pod <pod-name> | grep "Failed"
```

| Cause | Solution |
|-------|---------|
| Wrong image name | Fix the image name |
| Wrong tag | Check available tags |
| Private registry | Create imagePullSecret |
| Network issue | Check node internet access |

### 4. Service Has No Endpoints

```bash
kubectl get endpoints my-service
# NAME          ENDPOINTS
# my-service    <none>     ← Problem!
```

| Cause | Solution |
|-------|---------|
| Selector doesn't match pods | Fix selector labels |
| Pods not Ready | Check readiness probes |
| Pods in wrong namespace | Check namespace |

### 5. Deployment Stuck

```bash
kubectl rollout status deployment/my-app
kubectl rollout undo deployment/my-app    # Emergency rollback!
```

| Cause | Solution |
|-------|---------|
| New image failing | Rollback: `kubectl rollout undo` |
| Insufficient resources | Scale up nodes |
| PVC not bound | Fix storage issues |
| MaxSurge hit | Wait or increase surge |

---

## 🔧 Advanced Debugging

### Debug a Deleted Pod

```bash
# Get pod logs even after deletion
kubectl logs <pod-name> --previous

# Check events for why it was evicted
kubectl get events --field-selector reason=Killing
```

### Debug Networking

```bash
# Test DNS resolution
kubectl exec -it test-pod -- nslookup kubernetes.default

# Test service connectivity
kubectl exec -it test-pod -- curl http://my-service:80

# Check network policies
kubectl get networkpolicies -n <namespace>

# Check if kube-proxy is running
kubectl get pods -n kube-system -l k8s-app=kube-proxy
```

### Debug with Ephemeral Container

```bash
# Add a debug container to a running pod (K8s 1.23+)
kubectl debug -it <pod-name> --image=busybox --target=<container-name>

# Debug on a node (requires SSH access)
kubectl debug node/<node-name> -it --image=busybox
```

### Resource Usage

```bash
# Check pod resource usage (needs metrics-server)
kubectl top pods
kubectl top nodes

# Check resource limits
kubectl describe pod <pod-name> | grep -A10 "Limits"
```

---

## 📋 Best Practices

- ✅ **Always check Events first** — they tell you what K8s tried to do
- ✅ **Use `kubectl describe`** before anything else
- ✅ **Set up proper logging** in your applications
- ✅ **Use liveness/readiness probes** for automatic recovery
- ✅ **Keep a debugging checklist** for common issues
- ❌ **Don't randomly delete pods** — investigate first!
- ❌ **Don't ignore warnings** — they become errors
- ❌ **Don't forget about events in different namespaces**

---

## ❌ Common Mistakes

1. **Jumping to conclusions** 🤦
   > Always follow the systematic approach: Status → Describe → Logs → Shell

2. **Not checking events** 🤦
   > Events are the #1 debugging tool. They tell you exactly what happened.

3. **Checking only current logs** 🤦
   > Always check `--previous` for crashed containers.

4. **Ignoring namespace** 🤦
   > `kubectl get pods` only shows current namespace. Use `-A` to see all.

---

## 🎤 Interview Questions

1. **How do you debug a pod stuck in Pending?**
   > `kubectl describe pod` — check Events for scheduling failures (insufficient resources, node selector, taints).

2. **How do you debug a CrashLoopBackOff?**
   > Check `kubectl logs --previous` for the crash reason. Check `kubectl describe pod` for OOMKilled or other failures.

3. **What is the most useful debugging command?**
   > `kubectl describe pod` — it shows the Events section with detailed history.

4. **How do you access a shell inside a running pod?**
   > `kubectl exec -it <pod-name> -- /bin/bash`

---

## 📝 Summary

| Step | Command | What to Look For |
|------|---------|-----------------|
| 1 | `kubectl get pods` | Status column |
| 2 | `kubectl describe pod` | Events section 🏆 |
| 3 | `kubectl logs` | Application errors |
| 4 | `kubectl exec -it` | Network/connectivity |
| 5 | `kubectl get events` | Cluster-wide issues |

> *"Ravi, remember this debugging flow: get → describe → logs → exec. It solves 90% of issues. The Events section in `describe` is your best friend! 🔍"*
