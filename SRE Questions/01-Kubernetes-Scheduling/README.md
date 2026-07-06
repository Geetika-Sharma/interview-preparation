# 01 - Kubernetes Scheduling (Senior SRE Interview Guide)

---

## 📌 Table of Contents

- Why Kubernetes Scheduling matters
- Beginner Definitions
- Architecture Overview
- How Kubernetes Scheduling works internally
- What happens when a Pod is created
- Scheduling decision flow
- First production failure scenario (Pending Pods)

---

# 🎯 Why Kubernetes Scheduling Matters

In companies like Uber, Netflix, or Airbnb:

- Thousands of services run on Kubernetes
- Tens of thousands of Pods are created daily
- Clusters run across multiple AWS regions
- Resource pressure is constant (CPU, memory, network, storage)

### Why interviewers care:

If scheduling breaks:

- Deployments fail
- Autoscaling stalls
- Critical services stop scaling
- Incident response begins immediately

👉 Senior SREs are expected to:
- Diagnose scheduling failures in minutes
- Understand scheduler internals
- Fix production issues without guesswork

---

# 📖 Beginner Definitions (Must Know)

## Pod
Smallest deployable unit in Kubernetes.
A group of one or more containers that always run together.

---

## Node
A worker machine (VM/EC2 instance) where Pods run.

---

## Cluster
A group of nodes managed by Kubernetes.

---

## Scheduler
A control-plane component that decides:
> "Which Node should run this Pod?"

---

## Kubelet
Agent running on each Node.
It starts and manages Pods assigned to that Node.

---

## Container Runtime
Software that runs containers (e.g., containerd, Docker).

---

## Namespace
Logical isolation boundary inside a cluster.

---

## Resource Requests
Minimum CPU/Memory guaranteed for a Pod.

---

## Resource Limits
Maximum CPU/Memory a Pod can use.

---

## Taint
A rule on a Node that prevents Pods from being scheduled unless they explicitly tolerate it.

---

## Toleration
A Pod setting that allows it to run on a tainted Node.

---

## Node Selector
Simple rule that forces a Pod to run only on Nodes with specific labels.

---

## Node Affinity
Advanced version of nodeSelector with flexible rules.

---

## PVC (Persistent Volume Claim)
Request for storage made by a Pod.

---

## PV (Persistent Volume)
Actual storage resource in the cluster.

---

## ResourceQuota
Limits total resource usage per namespace.

---

## Cluster Autoscaler
Automatically adds/removes Nodes based on demand.

---

# 🏗️ Architecture Overview

```text
Developer
   |
   v
Kubernetes API Server
   |
   v
Scheduler (decides placement)
   |
   v
Node (Worker Machine)
   |
   v
Kubelet
   |
   v
Pod → Container Runtime → Running Containers
```
---
⚙️ # How Kubernetes Scheduling Works (Internals)

## When a Pod is created:

**Step 1:** API Server receives request
Pod spec is stored in etcd

**Step 2:** Scheduler picks unscheduled Pod
- Watches API server
- Finds Pods without assigned Node

**Step 3:** Filtering phase (Hard constraints)

Scheduler removes Nodes that cannot run the Pod:
- Not enough CPU
- Not enough memory
- Node taints not tolerated
- Node selector mismatch
- PVC not bound
- Node is NotReady
- Node is cordoned

**Step 4:** Scoring phase (Soft ranking)
- Remaining Nodes are ranked based on:
- Least loaded Node
- Best affinity match
- Zone balancing
- Topology spread constraints

**Step 5:** Binding phase
Scheduler assigns Pod → Node

**Step 6:** Kubelet takes over
- Pulls image
- Starts container
- Reports status back
---
# 🚨 What Happens When a Pod is Created
```
kubectl apply -f pod.yaml
        |
        v
API Server stores Pod
        |
        v
Scheduler tries to place Pod
        |
        v
IF node found:
    Pod → Running

IF no node found:
    Pod → Pending
```
# 📊 Scheduling Decision Flow
Pod Created
    |
    v
Check Node Capacity (CPU / Memory)
    |
    v
Check Taints & Tolerations
    |
    v
Check Node Selectors / Affinity
    |
    v
Check PVC Binding
    |
    v
Check ResourceQuota
    |
    v
Check Node Health (Ready / NotReady)
    |
    v
Score valid nodes
    |
    v
Assign best node

# 🧪 First Production Scenario

You are on-call at 2 AM.

## A deployment fails:

```
payment-api-7f4d9   Pending
payment-api-7f4e1   Pending
payment-api-7f4e7   Pending
```
Developers report:

"New version is not deploying."

### Step 1: First instinct of Senior SRE

Never guess.

Always inspect:
```
kubectl describe pod payment-api-7f4d9
```

### Step 2: Look at Events

Example output:
```
0/20 nodes are available:
20 Insufficient cpu
```
OR
```
node(s) had taint:
dedicated=database:NoSchedule
```
OR
```
node(s) didn't match node selector
```
### Step 3: Categorize the failure

All scheduling failures fall into 6 buckets:

**1. Resource shortage**
- CPU
- Memory
- Disk

**2. Node restrictions**
- Taints
- Cordoned nodes
- Node affinity mismatch

**3. Storage issues**
- PVC not bound
- StorageClass failure

**4. Policy restrictions**
- ResourceQuota exceeded
- LimitRange violation

**5. Autoscaling lag**
- Cluster too slow to add nodes

**6. Control plane issues**
- Scheduler unhealthy
- API latency

## 🧠 Senior SRE Thinking Pattern

**Junior Engineer:**
"Pod is pending. Maybe restart it?"

**Senior SRE:**
"Which scheduling constraint is blocking placement?"

**Principal Engineer:**
"Which layer of the scheduling decision pipeline failed, and why did capacity planning not catch it earlier?"

**📌 Key Takeaways**
- Pending = Pod not assigned to a Node
- Scheduler evaluates multiple constraints
- Failures are always deterministic, not random
- 90% of issues come from:
  - CPU / memory
  - taints
  - affinity rules
  - PVC
  - Always start with kubectl describe pod
