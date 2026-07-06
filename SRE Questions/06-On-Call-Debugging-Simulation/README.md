# 06 - On-Call Debugging Simulation (Senior SRE Interview Guide)

A production-grade simulation of real on-call incidents involving Kubernetes, AWS, networking, and cascading system failures.

---

# Table of Contents

- Introduction
- How On-Call Works at Scale
- Incident Response Framework
- Scenario 1: Pods Stuck in Pending (Cluster-wide)
- Scenario 2: Intermittent 5xx Spike (Service Degradation)
- Scenario 3: Cross-AZ Latency Explosion
- Scenario 4: Kafka Lag Crisis
- Scenario 5: Complete Service Blackout
- Decision-Making Under Pressure
- Debugging Mental Model
- Senior SRE Response Template
- Key Takeaways

---

# Introduction

On-call in large-scale systems is not about running commands.

It is about:

> Rapid classification of failure domains under uncertainty.

You are expected to:
- Identify blast radius in minutes
- Separate symptoms from root cause
- Stop system-wide amplification
- Restore service safely

---

# How On-Call Works at Scale

Typical flow:

    PagerDuty Alert
        ↓
    Identify severity (SEV1 / SEV2)
        ↓
    Determine blast radius
        ↓
    Check dashboards (metrics/logs)
        ↓
    Validate Kubernetes + AWS layers
        ↓
    Mitigate first, diagnose second
        ↓
    Prevent recurrence

---

# Incident Response Framework

## Step 1: Stabilize
Stop the bleeding.

## Step 2: Diagnose
Find root cause across layers.

## Step 3: Recover
Restore service safely.

## Step 4: Prevent
Fix systemic issue.

---

# Scenario 1: Cluster-Wide Pods Stuck in Pending

## Symptoms

- Hundreds of pods pending
- Deployments blocked
- No traffic impact yet

---

## First Check

    kubectl describe pod <pod>

Look at:
- Events section

---

## Likely Causes

- EC2 capacity exhaustion
- Over-constrained scheduling
- Node group failure
- Autoscaler delay

---

## Decision

- Scale node group manually
- Temporarily relax constraints
- Shift deployment traffic

---

# Scenario 2: Sudden 5xx Spike

## Symptoms

- API returning 500s
- Latency increasing
- Partial service failure

---

## Root Cause Categories

- Retry storm
- Downstream DB overload
- Pod crash loop
- Network degradation

---

## Debug Flow

    Check ingress metrics
    ↓
    Check pod logs
    ↓
    Check dependency latency
    ↓
    Check retry rate

---

## Fix

- Stop retry amplification
- Reduce traffic load
- Restart degraded service

---

# Scenario 3: Cross-AZ Latency Explosion

## Symptoms

- p99 latency spike
- Only cross-zone traffic affected
- Intra-zone traffic healthy

---

## Root Causes

- AWS inter-AZ congestion
- Load balancer imbalance
- VPC routing issues

---

## Fix

- Force zone-local traffic
- Rebalance load distribution
- Reduce cross-zone dependencies

---

# Scenario 4: Kafka Lag Crisis

## Symptoms

- Consumer lag increasing rapidly
- Event processing delayed
- Downstream systems stale

---

## Root Causes

- Consumer failure
- Partition imbalance
- Broker overload
- Processing slowdown

---

## Fix

- Scale consumers
- Increase partitions
- Reduce message processing time

---

# Scenario 5: Complete Service Blackout

## Symptoms

- Entire service unavailable
- Multiple subsystems failing
- Kubernetes + AWS both degraded

---

## Likely Causes

- Cascading retry storm
- Regional AWS degradation
- Node group collapse
- DNS failure (CoreDNS)

---

## Recovery Strategy

    Identify primary failure layer
        ↓
    Isolate affected dependency
        ↓
    Stop traffic amplification
        ↓
    Restore critical path first

---

# Decision-Making Under Pressure

Senior SREs do NOT:

- Randomly restart pods
- Guess root cause
- Jump between services

They DO:

- Classify failure domain quickly
- Reduce system load immediately
- Stabilize before deep debugging

---

# Debugging Mental Model

Think in layers:

    Application
        ↓
    Kubernetes (Pods / Nodes)
        ↓
    Networking (DNS / CNI / Service)
        ↓
    AWS (EC2 / EBS / VPC)
        ↓
    External dependencies

---

# Senior SRE Response Template

When I receive an on-call alert, I first assess the blast radius to determine whether the issue is isolated or system-wide. I then check Kubernetes-level signals such as pod status and events to identify scheduling, networking, or runtime issues. Next, I validate underlying AWS dependencies such as EC2 capacity, EBS health, and VPC networking. If the system is under active degradation, I prioritize mitigation actions such as stopping traffic amplification, scaling resources, or isolating faulty zones. Only after stabilizing the system do I proceed with deep root cause analysis and long-term remediation planning.

---

# Key Takeaways

- On-call = classification + stabilization first
- Root cause comes AFTER mitigation
- Most incidents are multi-layer failures
- Retry storms amplify small issues into outages
- Senior engineers prioritize system stability over diagnosis

---

# End of Module 06
