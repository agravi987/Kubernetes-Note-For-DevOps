# Kubernetes Notes - Learning Guide

> A structured, step-by-step Kubernetes study guide. Follow the topics in order for the best learning experience.

---

## How to Use These Notes

- Start from **Topic 00** and progress sequentially -- each topic builds on the previous one.
- Each folder contains a single `README.md` with concepts, examples, and command references.
- Use the **kubectl** commands in each note to practice alongside reading.
- Revisit earlier topics when something in a later topic feels unclear.

---

## Prerequisites

Before starting, make sure you have:

- A working terminal / command line
- Docker installed and running
- A Kubernetes cluster (Minikube, kind, or cloud-based)
- `kubectl` installed and configured

Start here: [00-Prerequisites](./00-Prerequisites/README.md)

---

## Topics

### Foundation

| # | Topic | Description |
|---|-------|-------------|
| 00 | [Prerequisites](./00-Prerequisites/README.md) | Tools and setup needed before you begin |
| 01 | [Introduction](./01-Introduction/README.md) | What is Kubernetes and why it matters |
| 02 | [Installation](./02-Installation/README.md) | Setting up a Kubernetes cluster |
| 03 | [kubectl](./03-kubectl/README.md) | The command-line tool for interacting with Kubernetes |
| 04 | [Architecture](./04-Architecture/README.md) | Control plane, nodes, and how Kubernetes works internally |
| 05 | [Labels and Selectors](./05-Labels-and-Selectors/README.md) | Metadata mechanism used to organize and filter resources |

### Core Workloads

| # | Topic | Description |
|---|-------|-------------|
| 06 | [Pods](./06-Pods/README.md) | The smallest deployable unit in Kubernetes |
| 07 | [ReplicaSets](./07-ReplicaSets/README.md) | Maintaining a desired number of pod replicas |
| 08 | [Deployments](./08-Deployments/README.md) | Managing rolling updates and rollbacks for applications |

### Networking and Access

| # | Topic | Description |
|---|-------|-------------|
| 09 | [Services](./09-Services/README.md) | Exposing pods to internal and external traffic |
| 10 | [Namespaces](./10-Namespaces/README.md) | Isolating resources within a cluster |

### Configuration

| # | Topic | Description |
|---|-------|-------------|
| 11 | [ConfigMaps](./11-ConfigMaps/README.md) | Managing non-sensitive configuration data |
| 12 | [Secrets](./12-Secrets/README.md) | Managing sensitive data like passwords and tokens |

### Storage

| # | Topic | Description |
|---|-------|-------------|
| 13 | [Volumes](./13-Volumes/README.md) | Attaching storage to pods |
| 14 | [Persistent Volumes and PVC](./14-Persistent-Volumes-and-PVC/README.md) | Cluster-level storage and dynamic provisioning |

### Advanced Networking

| # | Topic | Description |
|---|-------|-------------|
| 15 | [Ingress](./15-Ingress/README.md) | HTTP routing and load balancing for external access |
| 24 | [Network Policies](./24-Network-Policies/README.md) | Controlling traffic flow between pods |

### Specialized Workloads

| # | Topic | Description |
|---|-------|-------------|
| 16 | [StatefulSets](./16-StatefulSets/README.md) | Managing stateful applications with stable identities |
| 17 | [DaemonSets](./17-DaemonSets/README.md) | Running a pod on every node in the cluster |
| 18 | [Jobs and CronJobs](./18-Jobs-and-CronJobs/README.md) | Running one-off and scheduled tasks |

### Operations

| # | Topic | Description |
|---|-------|-------------|
| 19 | [Probes](./19-Probes/README.md) | Health checks: liveness, readiness, and startup |
| 20 | [Resource Management](./20-Resource-Management/README.md) | CPU and memory requests, limits, and quotas |
| 21 | [Horizontal Pod Autoscaler](./21-Horizontal-Pod-Autoscaler/README.md) | Automatically scaling pods based on metrics |
| 22 | [Scheduling](./22-Scheduling/README.md) | Controlling where pods get placed on nodes |

### Security

| # | Topic | Description |
|---|-------|-------------|
| 23 | [RBAC](./23-RBAC/README.md) | Role-Based Access Control for cluster permissions |

### Wrap-Up

| # | Topic | Description |
|---|-------|-------------|
| 25 | [Debugging](./25-Debugging/README.md) | Troubleshooting common issues in a cluster |
| 26 | [Best Practices](./26-Best-Practices/README.md) | Production-ready guidelines and patterns |
| 27 | [Interview Questions](./27-Interview-Questions/README.md) | Common Kubernetes interview questions and answers |

---

## Suggested Learning Path

```
Week 1:  Topics 00 - 05  (Foundation)
Week 2:  Topics 06 - 10  (Core Workloads + Networking)
Week 3:  Topics 11 - 15  (Configuration, Storage, Ingress)
Week 4:  Topics 16 - 20  (Specialized Workloads + Operations)
Week 5:  Topics 21 - 27  (Autoscaling, Security, Debugging, Interview Prep)
```

Adjust the pace based on your comfort level. Spend more time on Architecture (04), Pods (06), Deployments (08), and Services (09) -- these are the most important topics for interviews and day-to-day work.

---

## Quick Reference

| Task | Topic |
|------|-------|
| Create a pod | [06-Pods](./06-Pods/README.md) |
| Deploy an app | [08-Deployments](./08-Deployments/README.md) |
| Expose a service | [09-Services](./09-Services/README.md) |
| Store config data | [11-ConfigMaps](./11-ConfigMaps/README.md) |
| Store secrets | [12-Secrets](./12-Secrets/README.md) |
| Attach storage | [14-Persistent-Volumes-and-PVC](./14-Persistent-Volumes-and-PVC/README.md) |
| Set up HTTPS routing | [15-Ingress](./15-Ingress/README.md) |
| Check pod health | [19-Probes](./19-Probes/README.md) |
| Scale pods automatically | [21-Horizontal-Pod-Autoscaler](./21-Horizontal-Pod-Autoscaler/README.md) |
| Control access | [23-RBAC](./23-RBAC/README.md) |
| Debug issues | [25-Debugging](./25-Debugging/README.md) |

---

*Last updated: July 2026*
