# 💼 Jobs and CronJobs

> *"Ravi, most Kubernetes workloads run forever (Deployments, DaemonSets). But what about tasks that should run once and finish? Like a database migration, a batch report, or a nightly backup? That's what Jobs and CronJobs do!"*

---

## 🤔 What are Jobs and CronJobs?

### Job
Runs a pod that **completes a task and then stops**. It ensures the task finishes successfully.

### CronJob
Runs a Job on a **recurring schedule** (like cron in Linux).

```
Job:       "Run this once"     → Pod starts → Task completes → Pod done ✅
CronJob:   "Run this every 5 min" → Creates Job → Pod → Complete → Wait → Repeat
```

---

## 💡 Why Do We Need Them?

| Without Jobs | With Jobs |
|-------------|----------|
| Manual task execution 🤦 | Automated & tracked ✅ |
| No retry on failure 😰 | Automatic retries 🔄 |
| No scheduled runs 🤷 | Cron scheduling built-in ⏰ |
| Pod disappears when node dies | Guaranteed to complete 💪 |

---

## 📝 YAML Examples

### Job

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
spec:
  backoffLimit: 3              # Retry up to 3 times on failure
  ttlSecondsAfterFinished: 3600  # Auto-delete after 1 hour
  template:
    spec:
      containers:
        - name: migrate
          image: my-app:1.0
          command: ["python", "manage.py", "migrate"]
      restartPolicy: Never     # Don't restart completed pods
```

### CronJob

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: nightly-backup
spec:
  schedule: "0 2 * * *"       # Every day at 2:00 AM (cron syntax)
  jobTemplate:
    spec:
      backoffLimit: 2
      template:
        spec:
          containers:
            - name: backup
              image: backup-tool:1.0
              command: ["bash", "/scripts/backup.sh"]
          restartPolicy: Never
  concurrencyPolicy: Forbid    # Don't run concurrent backups
  successfulJobsHistoryLimit: 3  # Keep last 3 successful jobs
  failedJobsHistoryLimit: 3     # Keep last 3 failed jobs
```

---

## 🧠 Important Concepts

### Job Completion Modes

```yaml
spec:
  completionMode: NonIndexed   # Default — pods run independently

  # OR
  completionMode: Indexed      # Each pod gets a unique index (0, 1, 2...)
```

### CronJob Concurrency Policies

| Policy | Behavior |
|--------|----------|
| **Allow** | Allow concurrent Jobs (default) |
| **Forbid** | Skip new Job if previous still running |
| **Replace** | Cancel running Job, start new one |

### Cron Schedule Syntax

```
┌────── minute (0-59)
│ ┌────── hour (0-23)
│ │ ┌────── day of month (1-31)
│ │ │ ┌────── month (1-12)
│ │ │ │ ┌────── day of week (0-6, Sun=0)
│ │ │ │ │
* * * * *

Examples:
  "0 2 * * *"     → Every day at 2 AM
  "*/5 * * * *"   → Every 5 minutes
  "0 0 * * 0"     → Every Sunday at midnight
  "30 8 1 * *"    → 8:30 AM on the 1st of every month
```

---

## 📚 kubectl Commands

```bash
# Create a Job
kubectl apply -f job.yaml

# List jobs
kubectl get jobs

# Watch job status
kubectl watch jobs

# See pods created by job
kubectl get pods -l job-name=db-migration

# See job logs
kubectl logs -l job-name=db-migration

# Delete a CronJob
kubectl delete cronjob nightly-backup

# Manually trigger a CronJob (create a Job now)
kubectl create job manual-backup --from=cronjob/nightly-backup
```

---

## 📋 Best Practices

- ✅ **Set `backoffLimit`** to retry on failure
- ✅ **Set `ttlSecondsAfterFinished`** to auto-cleanup completed jobs
- ✅ **Set resource limits** on Job containers
- ✅ **Use `restartPolicy: Never`** for Jobs (most cases)
- ✅ **Use `concurrencyPolicy: Forbid`** for CronJobs that can't overlap
- ❌ **Don't forget `restartPolicy: Never/OnFailure`** (default `Always` will fail)
- ❌ **Don't leave completed jobs lying around** — they consume resources
- ❌ **Don't schedule critical backups without monitoring**

---

## ❌ Common Mistakes

1. **Forgetting `restartPolicy: Never` or `OnFailure`** 🔥
   > Jobs REQUIRE `restartPolicy: Never` or `OnFailure`. `Always` (default) will fail validation.

2. **Not cleaning up completed jobs** 🤦
   > Without `ttlSecondsAfterFinished`, completed pods stay forever. Set a TTL!

3. **Overlap in CronJobs** 🤦
   > If a backup takes 10 minutes but runs every 5 minutes, they'll overlap. Use `concurrencyPolicy: Forbid`.

4. **No backoffLimit** 🤦
   > Without it, a failing job only tries once. Set `backoffLimit: 3` for resilience.

---

## 🎤 Interview Questions

1. **What is a Job?**
   > A Kubernetes workload that runs a pod to completion. It ensures the task finishes successfully with optional retries.

2. **What is a CronJob?**
   > A Kubernetes resource that creates Jobs on a recurring schedule using cron syntax.

3. **What restartPolicy should Jobs use?**
   > `Never` or `OnFailure`. `Always` is invalid for Jobs.

4. **What is concurrencyPolicy in CronJobs?**
   > Controls whether new Jobs can run concurrently: Allow (default), Forbid (skip), or Replace (cancel running, start new).

5. **How do you manually trigger a CronJob?**
   > `kubectl create job manual-run --from=cronjob/<name>`

---

## 📝 Summary

| Concept | What to Remember |
|---------|-----------------|
| **Job** | Run once, complete, stop |
| **CronJob** | Run on schedule |
| **restartPolicy** | Must be Never or OnFailure |
| **backoffLimit** | Number of retries |
| **concurrencyPolicy** | Allow, Forbid, Replace |
| **TTL** | Auto-cleanup completed jobs |

> *"Ravi, Jobs and CronJobs are the 'set it and forget it' tools of Kubernetes. Database migrations, backups, reports — anything that runs once and finishes. Just remember: `restartPolicy: Never`! 💼"*
