# 🏥 Probes

> *"Ravi, how does Kubernetes know if your app is actually working? Not just running — but actually healthy and ready to serve traffic? PROBES! They're like regular health checkups for your containers. Critical for production reliability! 🏥"*

<div align="center">

| 📖 Reading Time | 🎯 Difficulty | 🏷️ Category |
|:---:|:---:|:---:|
| ~8 min | ⭐⭐ Intermediate | Operations |

</div>

---

## 🤔 What are Probes?

Probes are **health checks** that Kubernetes performs on containers to determine their state.

```
Kubernetes asks your container:
  ├── "Are you alive?"           → Liveness Probe
  ├── "Are you ready for traffic?" → Readiness Probe
  └── "Have you started up?"      → Startup Probe
```

---

## 💡 Why Do We Need Probes?

| Without Probes | With Probes |
|---------------|------------|
| Deadlock → container stays Running 😱 | Deadlock → K8s restarts the container 🔄 |
| App not ready → gets traffic → errors 😰 | App not ready → waits, then routes traffic ✅ |
| Slow startup → killed too early 🤦 | Startup probe → waits as long as needed ⏳ |
| Users see errors during deploy 🤕 | Only healthy pods get traffic 🎉 |

---

## 📚 Types of Probes

### 1. Liveness Probe ❤️

**"Is the container alive?"** If it fails, K8s **restarts** the container.

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 15    # Wait 15s after container starts
  periodSeconds: 10          # Check every 10s
  failureThreshold: 3        # Restart after 3 failures
```

**Use when:** Your app can hang/deadlock but won't recover on its own.

---

### 2. Readiness Probe ✅

**"Is the container ready to serve traffic?"** If it fails, K8s **removes the pod from Service endpoints**.

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
  failureThreshold: 3
```

**Use when:** Your app needs time to warm up, load data, or connect to databases before serving.

---

### 3. Startup Probe 🚀

**"Has the container finished starting?"** While active, it **disables liveness and readiness checks**.

```yaml
startupProbe:
  httpGet:
    path: /healthz
    port: 8080
  periodSeconds: 10
  failureThreshold: 30    # 30 × 10s = 5 minutes to start
```

**Use when:** Your app takes a long time to start (Java, large data loading).

---

## 📝 YAML Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: web-app
          image: my-app:1.0
          ports:
            - containerPort: 8080

          # Startup: wait up to 5 min for app to start
          startupProbe:
            httpGet:
              path: /healthz
              port: 8080
            periodSeconds: 10
            failureThreshold: 30

          # Liveness: restart if app hangs
          livenessProbe:
            httpGet:
              path: /healthz
              port: 8080
            periodSeconds: 10
            failureThreshold: 3

          # Readiness: remove from traffic if not ready
          readinessProbe:
            httpGet:
              path: /ready
              port: 8080
            periodSeconds: 5
            failureThreshold: 3
```

---

## 🧠 Important Concepts

### Probe Handlers

```yaml
# HTTP GET — checks if endpoint returns 2xx/3xx
httpGet:
  path: /healthz
  port: 8080
  httpHeaders:
    - name: X-Custom-Header
      value: probe

# TCP Socket — checks if port is open
tcpSocket:
  port: 3306

# Exec — runs a command; exit 0 = success
exec:
  command:
    - cat
    - /tmp/healthy
```

> 🚪😂 **Fun Fact / Joke:** A Liveness Probe without proper timeout is like your Indian mom checking if you're sleeping — she opens the door, closes it, opens it again, checks if you're really sleeping, and then checks one more time just to be sure. LET THE POD BREATHE!

### Probe Timing

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 15   # Wait before first check
  periodSeconds: 10         # How often to check
  timeoutSeconds: 3         # Timeout for each check
  successThreshold: 1       # How many successes = healthy
  failureThreshold: 3       # How many failures = unhealthy
```

### What Happens When Probes Fail

```
Liveness Probe fails:
  → K8s restarts the container
  → New container starts fresh
  → If it keeps failing → CrashLoopBackOff

Readiness Probe fails:
  → Pod removed from Service endpoints
  → No new traffic sent to this pod
  → When probe passes → traffic resumes

Startup Probe fails:
  → Container killed and restarted
```

---

## 📋 Best Practices

- ✅ **Always set readiness probes** — they prevent traffic to unready pods
- ✅ **Always set liveness probes** — they recover from deadlocks
- ✅ **Use startup probes** for slow-starting applications
- ✅ **Keep probe endpoints lightweight** — fast response, no heavy computation
- ✅ **Set reasonable timeouts** — don't let probes hang
- ❌ **Don't make liveness probe too aggressive** — you'll kill healthy pods
- ❌ **Don't use the same endpoint** for liveness and readiness if they have different needs
- ❌ **Don't forget probes entirely** — this is a production must!

---

## ❌ Common Mistakes

1. **No probes at all** 🔥
   > Without liveness probes, deadlocked containers stay running forever. Without readiness probes, traffic hits unready pods.

2. **Liveness probe too aggressive** 🤦
   ```yaml
   # ❌ Bad — restarts pods during normal high CPU
   periodSeconds: 1
   failureThreshold: 1

   # ✅ Good — more lenient
   periodSeconds: 10
   failureThreshold: 3
   ```

3. **Probe hits a heavy endpoint** 🤦
   > Probe endpoints should respond in milliseconds. Don't query a database in your health check!

4. **Forgetting startup probe for slow apps** 🤦
   > Java apps or ML models can take minutes to start. Without startup probe, liveness will kill them prematurely.

---

## 🎤 Interview Questions

1. **What are the three types of probes?**
   > Liveness (is it alive?), Readiness (is it ready for traffic?), Startup (has it finished starting?).

2. **What happens when a Liveness Probe fails?**
   > Kubernetes restarts the container.

3. **What happens when a Readiness Probe fails?**
   > The pod is removed from Service endpoints and stops receiving traffic.

4. **When would you use a Startup Probe?**
   > For applications that take a long time to start. It delays liveness/readiness checks until the app is fully started.

5. **What probe handlers are available?**
   > HTTP GET, TCP Socket, and Exec command.

---

## 📝 Summary

| Probe | Question | On Failure | Use Case |
|-------|---------|------------|----------|
| **Liveness** | Is it alive? | Restart container | Deadlock recovery |
| **Readiness** | Ready for traffic? | Remove from Service | Warm-up, temp overload |
| **Startup** | Finished starting? | Kill container | Slow-starting apps |

> *"Ravi, Probes are the difference between a fragile app and a resilient one. Without them, Kubernetes is flying blind. With them, it can detect and fix problems automatically. ALWAYS set probes in production! 🏥"*

---

## 🔗 Navigation

| ← Previous | Home | Next → |
|:----------:|:----:|:------:|
| **[← 18: Jobs & CronJobs](../18-Jobs-and-CronJobs/README.md)** | [📚 Home](../README.md) | **[20: Resource Management →](../20-Resource-Management/README.md)** |

> 💡 **Tip:** Open the next topic in a new tab so you can come back to this one anytime!
