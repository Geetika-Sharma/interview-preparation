# 01 - Kubernetes Scheduling (Senior SRE Interview Guide)

A production-focused guide to Kubernetes scheduling failures, debugging, and internals for Senior Site Reliability Engineer interviews ( Big scale systems: 40,000+ cores, multi-AZ EKS clusters, high-throughput microservices).

---

# Table of Contents

- Introduction
- Why Scheduling Matters at Scale
- Core Concepts
- Architecture Overview
- Scheduling Workflow
- What a Pending Pod Really Means
- First Debugging Principles
- Failure Categories Overview
- Key Takeaways

---

# Introduction

Kubernetes scheduling decides which Node will run a Pod.

At small scale, this feels invisible.

At large scale (thousands of nodes and tens of thousands of pods), scheduling becomes one of the most critical systems in production because:

- Pods fail to schedule daily under load
- Misconfigurations can block entire deployments
- Autoscaling delays directly impact production traffic
- Affinity / topology rules fragment cluster capacity

A Senior SRE is expected to:
- Diagnose scheduling issues quickly under pressure
- Understand scheduler decision-making internals
- Distinguish between configuration vs capacity vs infrastructure failures
- Prevent recurrence through system-level fixes

---

# Why Scheduling Matters at Scale

At companies running large Kubernetes clusters:

- Thousands of Pods may be Pending at any time
- Nodes are constantly fragmented across CPU / memory / zones
- Multi-region and multi-zone constraints reduce placement flexibility
- EC2 or node provisioning is not instantaneous
- One misconfigured rule can block an entire service rollout

Scheduling is often the **first bottleneck in the deployment pipeline**.

---

# Core Concepts

## Pod
Smallest deployable unit in Kubernetes. Represents one or more containers that run together.

## Node
A worker machine (VM or EC2 instance) where Pods are executed.

## Cluster
A group of Nodes managed by Kubernetes control plane.

## Scheduler
Control-plane component responsible for assigning Pods to Nodes.

## Kubelet
Agent running on each Node that starts and monitors containers.

## Container Runtime
Software that actually runs containers (containerd, CRI-O, etc.).

---

## Taint
A rule applied on a Node that prevents Pods from scheduling unless explicitly allowed.

## Toleration
A Pod-level configuration that allows it to bypass Node taints.

## Node Selector
A simple rule that forces Pods to run only on Nodes with specific labels.

## Node Affinity
A flexible version of nodeSelector that supports complex matching rules.

---

## PVC (Persistent Volume Claim)
A request for storage made by a Pod.

## PV (Persistent Volume)
The actual storage resource bound to a PVC.

## ResourceQuota
Limits total resource usage per namespace.

## Cluster Autoscaler
Component that adds or removes Nodes based on pending workload demand.

---

# Architecture Overview

Kubernetes scheduling flow:

    Developer
        |
        v
    API Server
        |
        v
    etcd (stores Pod spec)
        |
        v
    Scheduler
        |
        +--> Filter Nodes (hard constraints)
        |
        +--> Score Nodes (best fit selection)
        |
        v
    Bind Pod to Node
        |
        v
    Kubelet starts container

---

# Scheduling Workflow (Step-by-Step)

1. Pod is created via API Server
2. Pod specification is stored in etcd
3. Scheduler watches for unscheduled Pods
4. Scheduler filters Nodes based on constraints:
   - CPU availability
   - Memory availability
   - Node taints
   - Node affinity rules
   - Node selectors
   - PVC binding status
   - Node health conditions
5. Scheduler scores remaining Nodes
6. Best Node is selected
7. Pod is bound to Node
8. Kubelet pulls image and starts container

---

# What a Pending Pod Really Means

A Pod in `Pending` state means:

> Kubernetes has accepted the Pod but has not assigned it to any Node.

Important clarification:

- Pending is NOT a runtime failure
- It is a scheduling failure
- It always has a deterministic cause

---

# First Debugging Principles (Golden Rule)

Always start with:

    kubectl describe pod <pod-name>

Then immediately check:

    Events:

The Events section is the **single most important signal** for scheduling issues.

---

Secondary checks:

    kubectl get nodes
    kubectl top nodes
    kubectl get events
    kubectl get pvc
    kubectl describe quota

---

# Failure Categories Overview

All scheduling issues fall into these buckets:

## 1. Resource constraints
- CPU exhaustion
- Memory exhaustion

## 2. Node constraints
- Taints
- NotReady nodes
- Node selector mismatch

## 3. Storage constraints
- PVC not bound
- Storage class mismatch

## 4. Policy constraints
- ResourceQuota exceeded
- LimitRange blocking

## 5. Scheduling logic constraints
- Affinity rules
- Topology spread rules

## 6. Infrastructure constraints
- Autoscaler delay
- Node provisioning delays

---

# Key Takeaways

- Scheduling is deterministic, not random
- Pending always has a root cause
- Events are the most important debugging signal
- Most issues come from CPU, memory, affinity, or PVC
- Senior SREs think in “constraint systems”, not individual errors

---

# 01 - Kubernetes Scheduling (Part 2)

## Pods Stuck in Pending — Real Production Failure Modes (Senior SRE Deep Dive)

---

# Objective

In real production systems (<Company_name>-scale Kubernetes clusters), "Pending Pods" is not a single issue.

It is a **symptom of scheduling failure**, and every case maps to a specific constraint violation.

This section trains you to:
- Break down Pending Pods systematically
- Identify root causes in minutes
- Answer senior-level interview questions confidently

---

# Golden Debugging Rule

Always start with:

    kubectl describe pod <pod-name>

Then focus on:

    Events:

This is the scheduler’s exact explanation of why placement failed.

---

# Mental Model

A Pod is Pending because:

    NO node in the cluster satisfies ALL scheduling constraints

Constraints include:
- CPU
- Memory
- Taints
- Affinity rules
- Storage
- Quotas
- Node health

Your job is to find WHICH constraint failed.

---

# 1. Insufficient CPU

## Definition
No node has enough available CPU to satisfy the Pod's request.

---

## Why it happens
- Over-requested CPU in deployments
- Sudden traffic spike
- Poor capacity planning
- No buffer for burst workloads

---

## Symptoms
- Multiple Pods in Pending state
- High CPU usage on all nodes
- Scheduler reports CPU exhaustion

---

## Evidence

    0/20 nodes are available: 20 Insufficient cpu

---

## Commands

    kubectl top nodes
    kubectl top pods

---

## Fix
Immediate:
- Scale cluster nodes
- Reduce CPU requests
- Move non-critical workloads

Long-term:
- Enable Cluster Autoscaler / Karpenter
- Right-size workloads
- Introduce load testing

---

# 2. Insufficient Memory

## Definition
Memory requested by Pods exceeds available cluster memory.

---

## Why it happens
- Memory leaks
- JVM misconfiguration
- Underestimated usage patterns

---

## Symptoms
- Pods stuck Pending
- Node memory near saturation

---

## Evidence

    0/15 nodes are available: Insufficient memory

---

## Fix
- Increase node size
- Fix memory leaks
- Adjust memory requests

---

# 3. Node Taints Blocking Scheduling

## Definition
Nodes are intentionally restricted to specific workloads.

---

## Example

Node:

    dedicated=database:NoSchedule

Pod:
- Does not tolerate this taint

---

## Symptoms

    node(s) had taint

---

## Fix
- Add toleration to Pod
- Or remove taint (if misconfigured)

---

# 4. Missing Tolerations

## Definition
Pod is not allowed to run on tainted nodes.

---

## Why it happens
- New node pools introduced
- Platform policy updates
- Misconfigured deployments

---

## Fix

    tolerations:
      - key: "dedicated"
        operator: "Equal"
        value: "database"
        effect: "NoSchedule"

---

# 5. Node Selector Mismatch

## Definition
Pod requires node labels that do not exist in the cluster.

---

## Symptoms

    node(s) didn't match node selector

---

## Common causes
- Wrong region label
- Missing GPU node pool
- Incorrect instance type label

---

## Fix
- Fix Pod spec OR
- Fix node labels

---

# 6. Node Affinity Conflict

## Definition
Complex scheduling rules eliminate all eligible nodes.

---

## Why it happens
- Over-restrictive affinity rules
- Multi-zone constraints
- Incorrect label assumptions

---

## Fix
- Relax affinity rules
- Review topology constraints

---

# 7. PVC Not Bound

## Definition
Pod is waiting for storage volume binding.

---

## Symptoms
PVC stuck in Pending state.

---

## Root causes
- StorageClass misconfiguration
- Zone mismatch (EBS constraint)
- Volume limit reached

---

## Fix
- Fix StorageClass
- Ensure correct AZ alignment
- Pre-provision volumes if needed

---

# 8. ResourceQuota Exhaustion

## Definition
Namespace-level CPU/memory quota exceeded.

---

## Symptoms
New Pods rejected or stuck Pending.

---

## Fix
- Increase quota
- Clean unused workloads

---

# 9. LimitRange Misconfiguration

## Definition
Default resource limits prevent scheduling.

---

## Why it happens
- Overly strict defaults
- Missing requests in manifests

---

## Fix
- Update LimitRange policies
- Ensure proper defaults

---

# 10. Cluster Autoscaler Delay

## Definition
Cluster has capacity gap while new nodes are being provisioned.

---

## Why it happens
- EC2 boot time delay
- Scale-up lag
- Instance type unavailability

---

## Symptoms
- Temporary Pending state
- Then resolves automatically

---

## Fix
- Pre-warm nodes
- Use faster provisioning strategies (e.g., Karpenter)

---

# 11. Node NotReady

## Definition
Node exists but is not healthy enough to accept Pods.

---

## Causes
- Kubelet crash
- Network failure
- Disk issues

---

## Fix
- Restart kubelet
- Replace node
- Investigate system logs

---

# 12. Disk Pressure

## Definition
Node is low on disk space and rejects new Pods.

---

## Symptoms

    node.kubernetes.io/disk-pressure

---

## Fix
- Clean disk
- Increase storage
- Rotate logs

---

# 13. Topology Spread Constraint Failure

## Definition
Pods cannot be placed due to zone balancing rules.

---

## Why it happens
- Uneven capacity across AZs
- Strict spreading rules

---

## Fix
- Relax constraints
- Add capacity in missing zones

---

# 14. Priority / Preemption Conflicts

## Definition
Low-priority Pods cannot be scheduled due to higher-priority workloads.

---

## Fix
- Enable preemption
- Adjust priority classes
- Rebalance workloads

---

# Debugging Flow (Real Incident Approach)

    Pending Pod
        |
        v
    kubectl describe pod
        |
        v
    Read Events
        |
        +--> CPU / Memory issue
        +--> Taints issue
        +--> Affinity issue
        +--> PVC issue
        +--> Quota issue
        +--> Node issue
        +--> Autoscaler delay
        |
        v
    Validate cluster state
        |
        v
    Apply fix + verify recovery

---

# Senior SRE Thinking Pattern

Junior:
- Restart pod

Mid-level:
- Check CPU and memory

Senior:
- Identify scheduling predicate failure

Principal:
- Identify systemic capacity or design flaw causing scheduling constraints

---

# Interview Answer Template

When Pods are stuck in Pending, I start by checking `kubectl describe pod` and analyzing scheduler events. These events directly indicate which scheduling predicate failed. I categorize the issue into resource constraints, node constraints, storage constraints, policy constraints, or infrastructure constraints. Then I validate cluster state using node and metrics commands. Once I identify the root cause, I apply immediate mitigation (like scaling or configuration fixes) and then implement long-term prevention such as autoscaling, better resource requests, and improved capacity planning.

---

# Key Takeaways

- Pending Pods always have a deterministic cause
- Scheduler failure = constraint violation
- Events are the most important debugging signal
- Most failures fall into CPU, memory, affinity, or PVC issues
- Senior SREs think in systems, not individual errors

---

# 01 - Kubernetes Scheduling (Part 3)

## Scheduler Internals — How Kubernetes Actually Decides Where Pods Run

---

# Objective

In Part 1 and Part 2, we focused on **what breaks**.

Now we focus on:

> How Kubernetes scheduling actually works internally

This is where Senior / Staff-level interview questions come from.

---

# Why This Matters

At scale:
- Scheduling is not instant
- Pods are not placed randomly
- Failures are deterministic but multi-layered
- Multiple scheduler phases interact under load

Understanding internals helps you:
- Debug “invisible failures”
- Explain latency spikes
- Handle edge-case scheduling bugs

---

# Scheduler High-Level Model

Kubernetes scheduling happens in 3 phases:

    1. Filtering (Predicates)
    2. Scoring (Prioritization)
    3. Binding

---

# Architecture (Simplified)

    Pod Created
        |
        v
    Scheduler Watches API Server
        |
        v
    -------------------------
    |  Filtering Phase      |
    |  (Hard constraints)   |
    -------------------------
        |
        v
    -------------------------
    |  Scoring Phase        |
    |  (Best node choice)   |
    -------------------------
        |
        v
    Bind Pod to Node
        |
        v
    Kubelet starts container

---

# 1. Filtering Phase (Hard Constraints)

This phase eliminates nodes that CANNOT run the Pod.

A node is rejected if:

## CPU insufficient
## Memory insufficient
## Node taint not tolerated
## Node selector mismatch
## Node not ready
## PVC not bound
## Resource quota violated

---

## Key idea

Filtering is binary:

    Either node is valid OR rejected

No ranking happens here.

---

# 2. Scoring Phase (Soft Constraints)

After filtering, Kubernetes ranks nodes.

It prefers nodes that:

## Have more free resources
## Match affinity preferences
## Balance across zones
## Respect topology spread rules
## Reduce fragmentation

---

## Example scoring logic

    Node A: 80% free CPU → score 90
    Node B: 40% free CPU → score 50
    Node C: 70% free CPU → score 80

Scheduler selects Node A.

---

# 3. Binding Phase

Once a node is selected:

- Scheduler writes binding to API server
- Node assignment is finalized
- Kubelet pulls and runs container

---

# Preemption (Critical Concept)

Preemption happens when:

> A high priority Pod cannot be scheduled

Kubernetes may:
- Evict lower priority Pods
- Free resources
- Reschedule workloads

---

## Example

    High priority Pod arrives
    No nodes available

Kubernetes:
    Evicts low priority Pods
    Frees resources
    Schedules high priority Pod

---

# Priority Classes

Pods can have priority levels:

    system-critical > high > medium > low

This affects:
- Scheduling order
- Preemption decisions

---

# Scheduling Latency in Large Clusters

At scale, scheduling is delayed due to:

## 1. Too many Pods in queue
## 2. Large node filtering cost
## 3. Complex affinity rules
## 4. Cross-zone constraints
## 5. API server throttling

---

# Real Production Problem

Symptoms:

- Pods stuck Pending for 30–120 seconds
- No obvious resource issue
- Cluster is “not overloaded”

Root cause:

    Scheduler is spending too long evaluating node eligibility

---

# Why Scheduling Becomes Slow

Each Pod must evaluate:

- Every Node
- Every constraint
- Every label
- Every topology rule

At scale:

    5000 Pods × 1000 Nodes = huge evaluation cost

---

# Common Race Conditions

## 1. Autoscaler race

Pod scheduled → node not ready yet

---

## 2. PVC race

Pod scheduled → volume not attached yet

---

## 3. Node label update delay

Node updated → scheduler cache stale

---

## 4. Affinity conflict explosion

Too many constraints → zero valid nodes

---

# Topology Spread Constraints

Ensures Pods are evenly distributed:

Example:

    zone-a → 3 pods
    zone-b → 3 pods
    zone-c → 3 pods

If imbalance occurs → scheduling fails

---

# Real Failure Pattern

    Cluster has capacity
    BUT
    Topology rule prevents placement

This is a classic Senior SRE interview trap.

---

# Scheduler Cache Behavior

Scheduler does NOT query live node state every time.

It uses cached snapshots:

- Node list snapshot
- Resource snapshot
- Label snapshot

This can lead to:
- Slightly stale decisions
- Temporary scheduling inconsistencies

---

# Debugging Scheduler Internals

When debugging:

    kubectl get events -A

Check:
- Scheduling attempts
- Predicate failures
- Binding delays

---

# Senior SRE Mental Model

Junior:
- “Pod not scheduled, maybe cluster is full”

Mid:
- “Check CPU and memory”

Senior:
- “Which scheduling predicate failed during filtering?”

Principal:
- “Which part of scheduling pipeline is causing systemic inefficiency?”

---

# Interview Answer Template

Kubernetes scheduling works in three phases: filtering, scoring, and binding. In filtering, nodes that cannot satisfy hard constraints like CPU, memory, taints, affinity, and PVC are eliminated. In scoring, remaining nodes are ranked based on resource availability, topology balance, and affinity preferences. Finally, the scheduler binds the Pod to the selected node. In large clusters, scheduling delays are often caused by expensive filtering operations, complex affinity rules, or autoscaler latency. I debug by checking pod events, identifying which predicate failed, and validating cluster state using node and resource metrics.

---

# Key Takeaways

- Scheduling has 3 phases: Filter → Score → Bind
- Filtering is strict elimination
- Scoring is ranking
- Preemption allows eviction of lower priority Pods
- Most delays come from scale, not bugs
- Scheduler cache can cause slight inconsistencies

---

# Next Section

## Part 4 - Advanced Scheduling Edge Cases

We will cover:
- Real-world cascading scheduling failures
- Multi-zone outages
- Scheduler starvation scenarios
- Over-constrained clusters
- Large-scale production incidents (<Company_name>-level)
