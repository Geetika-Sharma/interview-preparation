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

# 01 - Kubernetes Scheduling (Part 4)

## Advanced Scheduling Edge Cases — Production Failures at Scale

---

# Objective

In real production systems (large EKS clusters like <Company_name>-scale infrastructure), scheduling failures are rarely simple.

They are often:
- Cascading
- Multi-factor
- Non-obvious
- Distributed across constraints

This section focuses on **real-world edge cases that senior engineers are expected to debug under pressure**.

---

# 1. Over-Constrained Cluster (Silent Failure Mode)

## Definition

A cluster where enough capacity exists in theory, but **no valid node satisfies all constraints simultaneously**.

---

## Why it happens

- Too many affinity rules
- Strict topology spread constraints
- Overuse of node selectors
- Tight taints across node groups

---

## Symptoms

- Nodes have free CPU and memory
- Pods remain Pending indefinitely
- No clear “resource exhausted” message

---

## Root Cause Pattern

    CPU OK
    Memory OK
    BUT
    No node satisfies ALL constraints together

---

## Fix

- Relax affinity rules
- Reduce topology constraints
- Remove unnecessary node selectors
- Review platform-level scheduling policies

---

# 2. Multi-Zone Fragmentation Problem

## Definition

Cluster capacity exists, but is unevenly distributed across availability zones.

---

## Example

    zone-a → full
    zone-b → full
    zone-c → empty capacity BUT wrong instance type

---

## Why it happens

- Uneven autoscaling across AZs
- Node group imbalance
- Traffic skew

---

## Symptoms

- Pods Pending only during scaling events
- Some AZs underutilized, others overloaded

---

## Fix

- Balance node groups per AZ
- Use topology-aware autoscaling
- Relax strict zone affinity

---

# 3. Scheduler Starvation (High Load Scenario)

## Definition

Some Pods never get scheduled because scheduler queue is overloaded.

---

## Why it happens

- Too many Pending Pods
- High churn in cluster
- Large-scale deployment rollouts

---

## Symptoms

- Scheduling delay increases gradually
- No explicit error in events
- Pods eventually schedule after long delay

---

## Root Cause

Scheduler is CPU-bound internally due to:
- Node evaluation cost
- Complex predicates
- Large cluster size

---

## Fix

- Reduce scheduling complexity
- Break deployments into batches
- Scale scheduler replicas (if supported)

---

# 4. Affinity Explosion Problem

## Definition

Too many affinity rules cause combinatorial filtering explosion.

---

## Example

Pod requires:
- zone = us-east-1a OR 1b
- instance-type = m6.large OR m6.xlarge
- node-label = gpu=true OR high-mem=true

---

## Result

Very few nodes satisfy all conditions simultaneously.

---

## Symptoms

- Sudden spike in Pending Pods
- No resource exhaustion errors
- Hard-to-debug scheduling failures

---

## Fix

- Simplify affinity rules
- Use soft preferences instead of hard requirements

---

# 5. PVC Zone Mismatch Failure

## Definition

Storage volumes are tied to specific AZs, but Pods are scheduled elsewhere.

---

## Why it happens

- EBS volumes are AZ-specific
- Pod affinity ignores storage locality

---

## Example

    PVC in zone-a
    Pod scheduled in zone-b → FAIL

---

## Symptoms

- Pending Pods with PVC errors
- Storage-related scheduling failure

---

## Fix

- Align node groups with storage zones
- Use volume binding topology

---

# 6. Node Pool Drift (Configuration Drift)

## Definition

Node labels and scheduling expectations diverge over time.

---

## Why it happens

- Manual node updates
- Auto-scaling group inconsistencies
- Platform configuration changes

---

## Symptoms

- NodeSelector mismatches
- Unexpected scheduling failures after deployments

---

## Fix

- Reconcile node labels
- Enforce infrastructure as code (Terraform)

---

# 7. Priority Inversion Scheduling Issue

## Definition

Lower priority Pods consume resources before higher priority Pods arrive.

---

## Why it happens

- No preemption enabled
- Burst workloads
- Improper priority class setup

---

## Symptoms

- High-priority Pods stuck Pending
- Cluster appears “full” even when it is not

---

## Fix

- Enable preemption
- Define proper priority classes

---

# 8. Scheduler Cache Staleness Issue

## Definition

Scheduler decisions are made using slightly outdated cluster state.

---

## Why it matters

At scale, even small delays cause:
- Incorrect placement decisions
- Temporary scheduling conflicts

---

## Symptoms

- Pods scheduled then immediately fail
- Inconsistent scheduling behavior

---

## Fix

- Increase scheduler sync frequency
- Reduce cluster churn

---

# 9. Cascading Scheduling Failure

## Definition

One scheduling issue triggers multiple downstream scheduling failures.

---

## Example

- Node group becomes NotReady
- Capacity drops suddenly
- Autoscaler reacts slowly
- Thousands of Pods become Pending

---

## Result

Full deployment degradation

---

## Fix

- Improve node health monitoring
- Faster autoscaling triggers
- Circuit-break deployment rollouts

---

# 10. Hidden Capacity (False Full Cluster)

## Definition

Cluster appears full due to fragmentation, not actual resource exhaustion.

---

## Why it happens

- Small unusable capacity blocks remain across nodes
- No single node can fit large requests

---

## Symptoms

- CPU available globally
- But no node can schedule large Pods

---

## Fix

- Enable bin-packing optimization
- Right-size workloads
- Use cluster rebalancing

---

# Debugging Flow (Advanced)

    Pending Pods Spike
            |
            v
    Check Events
            |
            v
    Is it resource issue?
        | YES → CPU/MEM
        |
        NO
        |
        v
    Check affinity / topology
        |
        v
    Check PVC / storage locality
        |
        v
    Check node pool health
        |
        v
    Check autoscaler delay
        |
        v
    Check scheduler saturation
        |
        v
    Identify systemic constraint conflict

---

# Senior SRE Mental Model

Junior:
- “Cluster is full”

Mid-level:
- “Check CPU and memory usage”

Senior:
- “Cluster is over-constrained due to scheduling policy interactions”

Principal:
- “This is a systemic capacity architecture problem, not a node-level issue”

---

# Interview Answer Template

At scale, Kubernetes scheduling failures are rarely due to simple resource exhaustion. Instead, they are caused by interactions between constraints such as affinity rules, topology spread, node selectors, storage locality, and autoscaling latency. I start by analyzing pod events to identify the scheduling predicate failure, then validate cluster state across nodes, zones, and storage layers. I categorize the issue into resource, policy, or topology constraints and determine whether the root cause is configuration, capacity fragmentation, or infrastructure delay. Finally, I apply immediate mitigation and design-level fixes such as relaxing constraints, improving autoscaling behavior, or restructuring node pools.

---

# Key Takeaways

- Most real failures are multi-factor, not single cause
- “Cluster full” often means “over-constrained”
- Scheduling issues scale non-linearly with cluster size
- Affinity + topology is a common hidden failure source
- Senior engineers think in system constraints, not symptoms

---

# 01 - Kubernetes Scheduling (Part 5)

## Production Incident Playbooks — Real On-Call Scenarios (Senior SRE Level)

---

# Objective

This section simulates **real production incidents** involving Kubernetes scheduling failures at scale.

These are the types of situations where:
- Services are down or degraded
- Deployments are blocked
- Autoscaling is not responding fast enough
- Multiple teams are paged simultaneously

You are expected to:
- Diagnose quickly under pressure
- Identify root cause across multiple systems
- Prevent recurrence, not just fix symptoms

---

# Incident 1: Massive Pending Pods During Deployment

## Symptoms

- 500+ Pods stuck in Pending
- Deployment rollout stuck
- No new version being released
- Alerts firing on deployment failure

---

## First Response (Senior SRE)

Always start with:

    kubectl describe pod <sample-pod>

Check:

    Events:

---

## Possible Root Causes

### 1. CPU Exhaustion Across Cluster
- Node capacity fully utilized
- No schedulable nodes exist

---

### 2. Over-restrictive Affinity Rules
- Deployment pinned to specific nodes
- No eligible nodes available

---

### 3. Node Pool Imbalance
- Some node groups full, others unused

---

### 4. Autoscaler Delay
- New nodes not yet provisioned

---

## Mitigation

- Scale cluster manually
- Relax deployment constraints temporarily
- Shift traffic to healthy services

---

## Long-term Fix

- Improve autoscaler thresholds
- Reduce affinity constraints
- Introduce capacity buffer strategy

---

# Incident 2: Multi-AZ Scheduling Failure

## Symptoms

- Pods stuck Pending only in specific regions
- Partial service outage
- Uneven traffic distribution

---

## Root Causes

### 1. AZ-specific capacity exhaustion
- One zone is full while others are underutilized

---

### 2. PVC zone binding restriction
- Volumes locked to specific AZ

---

### 3. Node group misalignment
- Some AZs missing correct instance types

---

## Fix

- Rebalance node groups per AZ
- Enable topology-aware scheduling
- Replicate storage across AZs

---

# Incident 3: Sudden Cluster "Full" Condition

## Symptoms

- No new Pods can be scheduled
- Existing Pods are healthy
- CPU usage is only ~60%

---

## Root Cause

This is a **fragmentation issue**, not resource exhaustion.

Example:

- Many small free spaces across nodes
- No single node fits requested Pod size

---

## Fix

- Enable bin packing optimization
- Reduce large resource requests
- Rebalance workloads

---

# Incident 4: PVC-Related Scheduling Outage

## Symptoms

- Pods stuck in Pending
- PVC status = Pending
- Deployment blocked

---

## Root Causes

### 1. StorageClass misconfiguration
### 2. AZ mismatch with EBS volume
### 3. Volume limit reached in region

---

## Fix

- Correct StorageClass
- Align node and volume AZ
- Pre-provision storage

---

# Incident 5: Priority Inversion Outage

## Symptoms

- Critical services not starting
- Lower priority workloads running normally
- Cluster appears healthy

---

## Root Cause

- No preemption enabled
- High-priority Pods blocked by low-priority workloads

---

## Fix

- Enable Pod preemption
- Adjust priority classes
- Evict low priority workloads

---

# Incident 6: Autoscaler Not Responding Fast Enough

## Symptoms

- Pods Pending for extended time
- Cluster eventually recovers
- Delay impacts user traffic

---

## Root Causes

- EC2 instance startup latency
- Scale-up threshold too conservative
- Image pull delays on cold nodes

---

## Fix

- Use faster provisioning (e.g., warm pools)
- Optimize autoscaler thresholds
- Pre-warm nodes for peak traffic

---

# Incident 7: Scheduler Bottleneck Under High Load

## Symptoms

- Scheduling latency increases
- Deployment delays observed
- No direct resource exhaustion

---

## Root Cause

- Scheduler CPU saturation
- Too many Pending Pods evaluated simultaneously
- Complex affinity rules increasing computation cost

---

## Fix

- Simplify scheduling rules
- Reduce Pod churn
- Scale scheduler replicas (where applicable)

---

# Incident 8: Cascading Scheduling Failure

## Symptoms

- One service failure spreads across system
- Multiple deployments fail simultaneously
- Cluster-wide Pending Pod explosion

---

## Root Cause Chain

1. Node group failure
2. Capacity reduction
3. Autoscaler delay
4. Scheduler backlog
5. System-wide scheduling congestion

---

## Fix

- Restore node group health
- Throttle deployments
- Stabilize autoscaler behavior

---

# Senior SRE Incident Response Flow

    Alert Triggered
          |
          v
    Identify Impact Scope
          |
          v
    Check Pending Pods
          |
          v
    kubectl describe pod
          |
          v
    Read Events
          |
          v
    Classify Issue:
        - Resource
        - Policy
        - Storage
        - Topology
        - Infrastructure
          |
          v
    Validate Cluster State
          |
          v
    Mitigate Immediately
          |
          v
    Apply Long-term Fix

---

# What Interviewers Are Testing

They are NOT testing Kubernetes commands.

They are testing:

- Can you classify complex failures quickly?
- Can you identify multi-system interactions?
- Can you handle ambiguity under pressure?
- Can you reason about distributed systems?

---

# Senior SRE Answer Template

In a production incident involving scheduling failures, I first determine the blast radius by checking the number of Pending Pods and affected services. I then inspect pod events using `kubectl describe pod` to identify scheduler-level failure reasons. I categorize the issue into resource constraints, policy constraints, storage constraints, topology constraints, or infrastructure-level failures. After validating cluster state across nodes, autoscaler, and storage systems, I apply immediate mitigation such as scaling nodes, adjusting constraints, or rerouting traffic. Finally, I implement long-term fixes such as improving autoscaling responsiveness, reducing scheduling complexity, and preventing fragmentation in node pools.

---

# Key Takeaways

- Scheduling failures are often system-wide incidents
- Root cause is often multi-layered, not single-node issues
- Fragmentation is as dangerous as resource exhaustion
- Autoscaling delay is a common real-world bottleneck
- Senior SREs focus on system stability, not just fixes

---

# End of Kubernetes Scheduling Module


# 01 - Kubernetes Scheduling (Part 6)

## Final Revision Sheet — Senior SRE Interview Mastery

---

# Objective

This is a **high-density revision layer** of everything covered in Kubernetes Scheduling.

Use this before interviews to quickly recall:
- Failure patterns
- Debugging flow
- Key commands
- Senior-level framing

---

# Core Truths About Kubernetes Scheduling

- Scheduling is deterministic
- Every failure has a constraint behind it
- “Pending” is never random
- Scheduler only makes decisions based on available cluster state
- Most real incidents are multi-factor, not single cause

---

# Golden Debug Command

    kubectl describe pod <pod-name>

Always check:

    Events:

This alone resolves 70% of interview scenarios.

---

# 6 Scheduling Constraint Groups

## 1. Resource Constraints
- CPU
- Memory

## 2. Node Constraints
- Taints
- Node readiness
- Node selectors

## 3. Storage Constraints
- PVC binding
- AZ mismatch

## 4. Policy Constraints
- ResourceQuota
- LimitRange

## 5. Topology Constraints
- Zone spreading
- Affinity rules

## 6. Infrastructure Constraints
- Autoscaler delay
- Scheduler saturation

---

# Top 12 Failure Patterns (Must Remember)

1. Insufficient CPU
2. Insufficient Memory
3. Node Taints
4. Missing Tolerations
5. Node Selector mismatch
6. Affinity conflict
7. PVC Pending
8. ResourceQuota exceeded
9. LimitRange blocking
10. Node NotReady
11. Disk Pressure
12. Topology spread failure

---

# Debugging Flow (Mental Model)

    Pending Pod
        |
        v
    Check Events (kubectl describe pod)
        |
        v
    Identify constraint type:
        - Resource?
        - Policy?
        - Storage?
        - Node?
        - Topology?
        - Infra?
        |
        v
    Validate cluster state
        |
        v
    Apply fix
        |
        v
    Prevent recurrence

---

# Key Commands (Must Know)

    kubectl get pods -A
    kubectl describe pod <pod>
    kubectl get nodes
    kubectl top nodes
    kubectl get events -A
    kubectl get pvc
    kubectl describe quota

---

# Senior vs Junior Thinking

## Junior
- Restart Pod
- Guess CPU issue

## Mid-level
- Check CPU / Memory metrics
- Look at node status

## Senior
- Identify scheduling predicate failure
- Validate multi-layer constraints
- Check system-wide capacity and topology

## Principal
- Identify systemic design flaw
- Fix architecture, not symptoms
- Optimize scheduling behavior at scale

---

# Interview “Perfect Answer” (Memorize)

When a Pod is stuck in Pending, I first inspect `kubectl describe pod` and analyze scheduler events, which directly indicate the failed scheduling predicate. I then classify the issue into resource constraints, node constraints, storage constraints, policy constraints, topology constraints, or infrastructure constraints. After identifying the category, I validate cluster state using node and metrics data. Once confirmed, I apply immediate mitigation such as scaling nodes or adjusting constraints, and then implement long-term fixes like improving autoscaling behavior, reducing scheduling complexity, and preventing cluster fragmentation.

---

# Common Interview Traps

- Assuming Pending = CPU issue (wrong)
- Ignoring affinity and topology constraints
- Not checking PVC binding issues
- Forgetting autoscaler delay scenarios
- Overlooking multi-zone scheduling failures

---

# High-Impact Insight

Most real-world scheduling failures are NOT:
- CPU shortage

They ARE:
- Constraint conflicts
- Over-restriction
- Fragmentation
- Delayed capacity provisioning

---

# Final Takeaways

- Scheduling is a constraint-solving system
- Pending = unsatisfied constraints, not errors
- Events are the most important signal
- Real failures are multi-dimensional
- Senior engineers think in systems, not symptoms

---

# End of Module 01

