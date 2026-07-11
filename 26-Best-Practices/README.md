# ✅ Best Practices

> *"Ravi, knowing Kubernetes features is one thing. Using them RIGHT is another. These best practices come from real-world production experience. Follow them and you'll avoid 90% of common pitfalls! 💪"*

---

## 🏗️ General Best Practices

### Use Declarative Configuration

```bash
# ❌ Imperative (bad for production)
kubectl run my-app --image=nginx

# ✅ Declarative (always preferred)
kubectl apply -f my-deployment.yaml
```

> Always use YAML files + `kubectl apply`. It's version-controllable, repeatable, and auditable.

### Label Everything

```yaml
metadata:
  labels:
    app: web
    env: production
    team: backend
    version: v1.0
```

> Good labels make filtering, debugging, and network policies SO much easier.

### Use Namespaces

```
Don't dump everything in default namespace!
├── production
├── staging
├── development
└── monitoring
```

---

## 📦 Pod Best Practices

- ✅ **Never create bare Pods** — always use Deployments
- ✅ **Always set resource requests and limits**
- ✅ **Always set liveness and readiness probes**
- ✅ **Use non-root user** inside containers
- ✅ **Use specific image tags** — never `:latest`
- ✅ **Set `imagePullPolicy: IfNotPresent`** for tagged images
- ❌ **Don't run as root** (unless absolutely necessary)
- ❌ **Don't store data in container filesystem**

```yaml
# Good pod spec
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
  containers:
    - name: app
      image: my-app:1.2.3          # Pinned version
      imagePullPolicy: IfNotPresent
      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"
        limits:
          cpu: "500m"
          memory: "256Mi"
      livenessProbe:
        httpGet:
          path: /healthz
          port: 8080
      readinessProbe:
        httpGet:
          path: /ready
          port: 8080
```

---

## 🚀 Deployment Best Practices

- ✅ **Use RollingUpdate strategy** (default) for zero-downtime
- ✅ **Set `maxSurge: 1, maxUnavailable: 0`** for maximum availability
- ✅ **Use `kubectl rollout status`** during deployments
- ✅ **Keep revision history** for quick rollbacks
- ✅ **Test rollbacks** before you need them

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```

---

## 🌐 Networking Best Practices

- ✅ **Use Services** for all inter-pod communication
- ✅ **Use Ingress** for external HTTP/HTTPS access
- ✅ **Apply default-deny Network Policies** in production
- ✅ **Use DNS names** instead of IPs (Services give you DNS)
- ❌ **Don't hardcode Pod IPs** — they change!
- ❌ **Don't use NodePort in production** — use Ingress

---

## 🔐 Security Best Practices

### RBAC

- ✅ Follow **least-privilege** principle
- ✅ Use **dedicated ServiceAccounts** for applications
- ❌ Never give `cluster-admin` to applications

### Secrets

- ✅ **Enable encryption at rest** for etcd
- ✅ Use **external secret managers** (Vault, AWS SM) in production
- ❌ Never commit secrets to Git
- ❌ Never use base64 as encryption

### Container Security

```yaml
securityContext:
  runAsNonRoot: true
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false
  capabilities:
    drop:
      - ALL
```

---

## 💾 Storage Best Practices

- ✅ Use **PVC** for all persistent data
- ✅ Set **reclaimPolicy: Retain** for important data
- ✅ Use **StorageClasses** for dynamic provisioning
- ✅ **Backup PVs** regularly
- ❌ Don't use `emptyDir` for important data
- ❌ Don't use `hostPath` in production

---

## 📊 Resource Management Best Practices

- ✅ **Always set requests AND limits** on every container
- ✅ **Use LimitRange** to set default values per namespace
- ✅ **Use ResourceQuota** to prevent resource abuse
- ✅ **Set requests close to actual usage**
- ✅ **Use HPA** for automatic scaling

---

## 🔍 Monitoring & Debugging Best Practices

- ✅ **Set up centralized logging** (Fluentd, Loki)
- ✅ **Monitor node and pod metrics** (Prometheus + Grafana)
- ✅ **Set up alerts** for critical issues
- ✅ **Use `kubectl describe`** as first debugging step
- ✅ **Check Events** before anything else

---

## 📁 Production Checklist

```
Before deploying to production, verify:

□ Resource requests and limits set
□ Liveness and readiness probes configured
□ Rolling update strategy configured
□ Health check endpoints implemented
□ Non-root container user
□ Specific image tags (not :latest)
□ Network policies applied
□ RBAC configured (least privilege)
□ Secrets encrypted at rest
□ Monitoring and logging in place
□ Backup strategy for persistent data
□ ResourceQuota on namespace
□ PodDisruptionBudget for critical apps
```

> 😄 **Joke:** Best practices are just accumulated wisdom from other people's mistakes. In Indian terms — your mom already told you not to touch the pressure cooker whistle. Learn from her, don't be the reason everyone else learned! 👩‍🍳😂

---

## 🎤 Interview Questions

1. **What are the most important Kubernetes best practices?**
   > Always use Deployments, set resource limits, use probes, use declarative configs, follow least-privilege RBAC, and use namespaces.

2. **Why should you never use `:latest` tag in production?**
   > It's unpredictable — you don't know which version is deployed. Always pin specific versions.

3. **What is a PodDisruptionBudget?**
   > Ensures a minimum number of pods are always running during voluntary disruptions (node maintenance, upgrades).

---

## 📝 Summary

| Category | Key Practice |
|----------|-------------|
| **Config** | Declarative YAML + version control |
| **Pods** | Resources, probes, non-root, pinned images |
| **Deployments** | Rolling updates, revision history |
| **Security** | Least-privilege RBAC, no secrets in Git |
| **Storage** | PVC with Retain policy, regular backups |
| **Network** | Default-deny Network Policies |
| **Monitoring** | Centralized logs, metrics, alerts |

> *"Ravi, best practices are just accumulated wisdom from other people's mistakes. Follow them and you'll sleep better at night! 😴"*
