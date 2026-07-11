# 🎤 Interview Questions

> *"Ravi, this is the grand finale! Here are the most commonly asked Kubernetes interview questions, organized by topic. Use this as a revision sheet before your interviews. You've got this! 💪"*

---

## 🏗️ Architecture

1. **What are the components of the Kubernetes Control Plane?**
   > API Server, etcd, Scheduler, and Controller Manager.

2. **What does the API Server do?**
   > Front door for all communication. Handles authentication, authorization, validation, and stores state in etcd.

3. **What is etcd?**
   > Distributed key-value store. The single source of truth for all cluster data.

4. **What does kubelet do?**
   > Agent on each node that manages pods, starts/stops containers, runs health checks, and reports node status.

5. **What does kube-proxy do?**
   > Maintains network rules on each node for Service-based routing and load balancing.

6. **What is the reconciliation loop?**
   > Constant comparison of desired state vs actual state, with corrective action when they differ.

---

## 📦 Pods & Workloads

7. **What is a Pod?**
   > Smallest deployable unit in K8s. Wraps 1+ containers sharing network, storage, and lifecycle.

8. **What is a Deployment?**
   > Manages ReplicaSets, provides rolling updates, rollbacks, and scaling for stateless applications.

9. **What is a ReplicaSet?**
   > Ensures a specified number of pod replicas are running. Managed by Deployments.

10. **What is a StatefulSet?**
    > Workload for stateful applications. Provides stable network identities, persistent storage, and ordered deployment.

11. **What is a DaemonSet?**
    > Ensures one pod runs on every (or selected) node. Used for logging, monitoring, networking.

12. **What is a Job/CronJob?**
    > Job runs a pod to completion. CronJob creates Jobs on a schedule.

---

## 🌐 Networking

13. **What is a Service?**
    > Stable network endpoint (fixed IP + DNS) that load-balances traffic across pods.

14. **What are the Service types?**
    > ClusterIP (internal), NodePort (external via node), LoadBalancer (external via cloud LB), ExternalName (external DNS).

15. **What is Ingress?**
    > Routes external HTTP/HTTPS traffic to services based on host/path rules. Requires an Ingress Controller.

16. **What are Network Policies?**
    > Firewall rules controlling pod-to-pod communication. Default is allow-all; use default-deny in production.

17. **What is the difference between ClusterIP and NodePort?**
    > ClusterIP is internal-only. NodePort exposes on each node's IP at a static port (30000-32767).

---

## 🔐 Security

18. **What is RBAC?**
    > Role-Based Access Control. Defines who can do what on which resources.

19. **What is the difference between Role and ClusterRole?**
    > Role is namespace-scoped. ClusterRole is cluster-scoped.

20. **What are Kubernetes Secrets?**
    > Objects storing sensitive data (passwords, keys). Base64 encoded by default, NOT encrypted.

21. **How do you secure Secrets in production?**
    > Enable encryption at rest, use RBAC to restrict access, use external secret managers.

---

## 💾 Storage

22. **What is a PersistentVolume (PV)?**
    > Cluster-level storage resource provisioned by admin or dynamically.

23. **What is a PersistentVolumeClaim (PVC)?**
    > Request for storage. Binds to a matching PV based on size, access mode, and storage class.

24. **What is Dynamic Provisioning?**
    > Automatic PV creation when a PVC is submitted, using a StorageClass provisioner.

25. **What are the access modes?**
    > ReadWriteOnce (one node), ReadOnlyMany (many nodes read), ReadWriteMany (many nodes R/W).

---

## ⚙️ Configuration

26. **What is a ConfigMap?**
    > Stores non-sensitive configuration data. Injected as env vars or mounted as files.

27. **What is the difference between ConfigMap and Secret?**
    > ConfigMap is for non-sensitive data (plain text). Secret is for sensitive data (base64, can be encrypted).

---

## 📊 Scaling & Resources

28. **What is HPA?**
    > Horizontal Pod Autoscaler. Automatically scales pod replicas based on CPU/memory metrics.

29. **What is the difference between requests and limits?**
    > Requests = minimum guaranteed resources (used for scheduling). Limits = maximum allowed.

30. **What happens when a pod exceeds its memory limit?**
    > OOMKilled — the container is terminated by the kernel.

---

## 🏥 Health & Lifecycle

31. **What are the three types of probes?**
    > Liveness (restarts deadlocked pods), Readiness (removes unready pods from Service), Startup (delays checks for slow-starting apps).

32. **What does CrashLoopBackOff mean?**
    > Container repeatedly crashes and restarts. Backoff increases: 10s → 20s → 40s → up to 5min.

---

## 🔍 Debugging

33. **How do you debug a pod stuck in Pending?**
    > `kubectl describe pod` — check Events for scheduling failures.

34. **What is the most useful debugging command?**
    > `kubectl describe pod` — the Events section tells you everything.

35. **How do you check pod logs from a crashed container?**
    > `kubectl logs <pod-name> --previous`

---

## 🧠 Scenario-Based Questions

36. **Your pod is in CrashLoopBackOff. What do you check?**
    > 1. `kubectl logs --previous` (crash reason)
    > 2. `kubectl describe pod` (OOMKilled, image issues)
    > 3. Check ConfigMaps/Secrets exist
    > 4. Check resource limits

37. **Your service has no endpoints. What's wrong?**
    > 1. Selector doesn't match pod labels
    > 2. Pods are not in Ready state
    > 3. Wrong namespace

38. **How do you perform a zero-downtime deployment?**
    > Use Deployment with RollingUpdate strategy, set `maxSurge: 1, maxUnavailable: 0`, configure readiness probes.

39. **How would you restrict which pods can communicate?**
    > Network Policies. Start with default-deny, then add specific allow rules.

40. **A new pod is stuck in ImagePullBackOff. What do you check?**
    > 1. Image name and tag are correct
    > 2. Image exists in the registry
    > 3. imagePullSecrets configured (for private registries)
    > 4. Node has internet access

---

## 📝 Quick Reference

| Topic | Key Command |
|-------|------------|
| **Pod status** | `kubectl get pods` |
| **Pod details** | `kubectl describe pod <name>` |
| **Pod logs** | `kubectl logs <name> --previous` |
| **Shell into pod** | `kubectl exec -it <name> -- bash` |
| **Rollback** | `kubectl rollout undo deployment/<name>` |
| **Rollout status** | `kubectl rollout status deployment/<name>` |
| **Scale** | `kubectl scale deployment/<name> --replicas=5` |
| **Resource usage** | `kubectl top pods` |
| **Events** | `kubectl get events --sort-by=.metadata.creationTimestamp` |

---

## 💡 Tips for Kubernetes Interviews

1. **Always mention "desired state"** — it's the core concept of K8s
2. **Know the difference** between Deployment, ReplicaSet, and Pod
3. **Mention zero-downtime** when talking about rolling updates
4. **Talk about RBAC** when discussing security
5. **Explain the reconciliation loop** — interviewers LOVE this
6. **Give specific kubectl commands** in your answers
7. **Mention production considerations** (resource limits, probes, Network Policies)
8. **Know at least one CNI plugin** (Calico, Cilium)
9. **Understand PV/PVC/StorageClass** relationship
10. **Practice explaining concepts simply** — if you can explain it to a beginner, you truly understand it

> 😄 **Joke:** Pro tip for K8s interviews: when asked a question you don't know, just say "It depends on the use case" — technically correct, the best kind of correct. Works better than saying "my laptop doesn't support Kubernetes" 🎤😄

> *"Ravi, you've covered ALL the fundamentals now. The key to a great Kubernetes interview is not memorizing — it's understanding. Know the WHY behind each concept, and you'll answer any question they throw at you. Go crush that interview! 🎤"*

---

## 🔗 Navigation

| ← Previous | Home | Finish 🎉 |
|:----------:|:----:|:------:|
| **[← 26: Best Practices](../26-Best-Practices/README.md)** | [📚 Home](../README.md) | **[🏠 Back to Home](../README.md)** |

> 🎉 **Congratulations, Ravi!** You've completed the entire Kubernetes learning path! Go ace that interview! 🚀
