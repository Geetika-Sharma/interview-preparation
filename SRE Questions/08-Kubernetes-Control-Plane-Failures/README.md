# 08 - Kubernetes Control Plane Failures (etcd, API Server, Scheduler Internals)

A senior-level deep dive into how Kubernetes *actually breaks internally* at scale (40k+ cores, thousands of nodes, high QPS clusters).

---

# Table of Contents

- Introduction
- What the Kubernetes Control Plane Actually Is
- etcd (The Brain of the Cluster)
- API Server Failures
- Controller Manager Failures
- Scheduler Failures (Deeper Than Before)
- Watch Cache & Event Explosion Problems
- Control Plane Throttling at Scale
- Real Production Incident Patterns
- Debugging Flow
- Senior SRE Mental Model
- Interview Answer Template
- Key Takeaways

---

# Introduction

Most engineers debug Kubernetes at the **Pod level**.

Senior SREs debug Kubernetes at the:

> Control plane level (the system that *decides everything*)

If the control plane is degraded:
- Pods don’t schedule
- Deployments stall
- Nodes appear “healthy” but cluster is unusable

---

# Kubernetes Control Plane Overview

The control plane consists of:

- **API Server** → entry point for all requests
- **etcd** → source of truth (state database)
- **Scheduler** → assigns pods to nodes
- **Controller Manager** → reconciles desired state

---

# 1. etcd (The Brain of Kubernetes)

## Definition

etcd is a **distributed key-value store** that stores all cluster state.

Everything depends on it.

---

## What it stores

- Pod definitions
- Node status
- Deployments
- ConfigMaps
- Secrets
- Service state

---

## etcd Failure Modes

### 1. High latency writes

Causes:
- disk I/O saturation
- slow SSD / network disk

Symptoms:
- kubectl commands slow
- deployments hang

---

### 2. Leader election instability

Causes:
- network partition
- CPU pressure

Symptoms:
- API server timeouts
- cluster instability

---

### 3. Disk pressure / compaction issues

Causes:
- large state history
- no compaction configured

Symptoms:
- exponential latency increase

---

## Key Insight

> If etcd is slow → everything in Kubernetes is slow

---

# 2. API Server Failures

## Definition

API server is the **front door of Kubernetes**

Everything goes through it:
- kubectl
- controllers
- scheduler

---

## Failure Modes

### 1. Request throttling

Causes:
- too many watch connections
- high QPS from controllers

Symptoms:
- kubectl timeouts
- delayed updates

---

### 2. Connection saturation

Causes:
- too many clients (services, controllers, CI/CD)

Symptoms:
- intermittent failures

---

### 3. API server CPU overload

Causes:
- large cluster object count
- frequent updates

---

## Real Insight

> API server bottleneck is often the FIRST scaling limit of Kubernetes

---

# 3. Controller Manager Failures

## Definition

Controller Manager ensures cluster state matches desired state.

---

## What it does

- Deployments → ReplicaSets
- Nodes → health reconciliation
- Jobs → lifecycle management

---

## Failure Modes

### 1. Reconciliation lag

Causes:
- too many objects
- slow API server

Symptoms:
- Pods not recovering
- stale state

---

### 2. Crash loops

Causes:
- bad config
- API overload

---

## Key Insight

> If controller manager lags → cluster becomes “stuck in wrong state”

---

# 4. Scheduler Deep Failures

(We previously covered scheduling logic — now control-plane perspective)

---

## Failure Modes

### 1. Scheduling backlog

Causes:
- too many Pending pods
- slow predicate evaluation

Symptoms:
- increasing Pending queue

---

### 2. Cache desync

Causes:
- API delay
- node state mismatch

---

### 3. Scheduling starvation

Causes:
- complex affinity rules
- uneven workload distribution

---

## Key Insight

> Scheduler is not broken — it is overwhelmed

---

# 5. Watch Cache Explosion Problem

## Definition

Kubernetes uses "watch streams" to track changes.

---

## Failure Mode

- too many objects
- too many watch connections
- event flood

---

## Symptoms

- API server CPU spikes
- delayed updates
- inconsistent cluster state

---

## Root Cause

> Too many microservices watching too many objects

---

# 6. Control Plane Throttling

## Definition

Kubernetes limits API request rates.

---

## Failure Mode

- CI/CD pipelines hammer API server
- controllers retry aggressively
- exponential load increase

---

## Symptoms

- 429 errors (Too Many Requests)
- deployment delays
- slow cluster response

---

# 7. Real Production Incident Patterns

---

## Pattern 1: “Cluster is healthy but nothing works”

Cause:
- API server overloaded

---

## Pattern 2: “Deployments stuck everywhere”

Cause:
- etcd latency or controller lag

---

## Pattern 3: “Pods not scheduling even with free capacity”

Cause:
- scheduler backlog or cache mismatch

---

## Pattern 4: “Everything is slow at once”

Cause:
- control plane saturation (not worker nodes)

---

# 8. Debugging Flow (Senior Level)

    Step 1: Check API server health
        - latency
        - error rate

    Step 2: Check etcd health
        - latency
        - leader status

    Step 3: Check controller manager
        - reconciliation lag

    Step 4: Check scheduler queue
        - pending pods

    Step 5: Correlate control plane load

---

# 9. Senior SRE Mental Model

Junior:
- “Pod is broken”

Mid-level:
- “Node is unhealthy”

Senior:
- “Control plane is degraded”

Principal:
- “Kubernetes is hitting architectural scaling limits in etcd/API/scheduler layer”

---

# 10. Interview Answer Template

When diagnosing Kubernetes-wide issues, I first determine whether the control plane is degraded by checking API server latency, error rates, and etcd health. Since etcd is the source of truth, any degradation there affects the entire cluster. I then validate controller manager and scheduler behavior to identify reconciliation or scheduling lag. If the issue is widespread, I suspect control plane saturation due to high object counts, excessive watch connections, or request throttling. I prioritize stabilizing the control plane by reducing load, scaling components if possible, and isolating noisy workloads before performing deeper root cause analysis.

---

# 11. Key Takeaways

- etcd is the most critical dependency in Kubernetes
- API server is the bottleneck at scale
- Controllers define system correctness
- Scheduler is often overloaded, not broken
- Watch streams can silently kill control plane performance

---

# End of Module 08
