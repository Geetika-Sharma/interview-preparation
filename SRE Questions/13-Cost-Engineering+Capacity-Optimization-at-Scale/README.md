# 13 - Cost Engineering + Capacity Optimization at Scale (Senior SRE Interview Guide)

A production-grade deep dive into how large-scale infrastructure teams (Kubernetes + AWS + microservices) control cost, improve utilization, and prevent silent capacity waste at 10k+ nodes / 40k+ cores scale.

---

# Table of Contents

- Introduction
- Why Cost Engineering is an SRE Problem
- Core Cost Drivers in Kubernetes + AWS
- Cluster Utilization Fundamentals
- Bin Packing & Fragmentation Problems
- Overprovisioning in Microservices
- Autoscaling (HPA / VPA / Cluster Autoscaler)
- Rightsizing EC2 and Node Groups
- Kafka / DB Cost Amplification
- Cost Anomaly Detection Systems
- Multi-Account AWS Cost Governance
- Real Production Incident Patterns (Cost Failures)
- Debugging Flow (Cost + Capacity Issues)
- Senior SRE Mental Model
- Interview Answer Template
- Key Takeaways

---

# Introduction

At scale, infrastructure cost is NOT just finance concern.

It becomes:

> a reliability problem disguised as a budgeting problem

Because wasted capacity leads to:
- unstable autoscaling
- inefficient scheduling
- unexpected outages during spikes
- hidden fragility under load

---

# Why Cost Engineering is an SRE Problem

Cost inefficiency causes:

- node overprovisioning → wasted capacity
- underutilization → scheduling fragmentation
- wrong autoscaling → delayed scaling during spikes
- noisy neighbor effects

So:

> cost = reliability + efficiency + scaling headroom

---

# Core Cost Drivers in Kubernetes + AWS

## AWS Layer

- EC2 instances
- EBS volumes
- NAT gateways (very expensive at scale)
- Load balancers
- Data transfer costs

---

## Kubernetes Layer

- idle pods
- over-requested CPU/memory
- poor bin packing
- unnecessary replicas

---

## Data Systems

- Kafka over-retention
- database over-provisioning
- excessive replication

---

# Cluster Utilization Fundamentals

## Definition

How efficiently CPU + memory is used across nodes.

---

## Key Problem

Even at 60% average utilization:

> cluster can still be 30–40% inefficient due to fragmentation

---

# Bin Packing & Fragmentation Problems

## Definition

Workloads do not fit efficiently into nodes.

---

## Causes

- large uneven resource requests
- too many small services
- strict node affinity rules

---

## Symptoms

- nodes partially filled but cannot schedule new pods
- autoscaler scales unnecessarily
- high cost with low CPU pressure

---

## Fix

- right-size requests/limits
- consolidate workloads
- adjust scheduling constraints
- use VPA recommendations

---

# Overprovisioning in Microservices

## Definition

Services request more CPU/memory than they actually use.

---

## Why it happens

- safety margins
- unknown traffic patterns
- lack of profiling
- copy-paste deployment configs

---

## Symptoms

- low CPU utilization clusters
- frequent unnecessary scaling
- high AWS bill with stable performance

---

## Fix

- enforce resource audits
- use profiling tools
- implement request enforcement policies

---

# Autoscaling Systems

## 1. HPA (Horizontal Pod Autoscaler)

- scales pods based on CPU/memory/metrics

---

## 2. VPA (Vertical Pod Autoscaler)

- adjusts resource requests dynamically

---

## 3. Cluster Autoscaler

- adds/removes nodes based on pending pods

---

## Common Failure Modes

### 1. Slow scaling response
### 2. Oscillation (scale up/down loops)
### 3. Overreaction to transient spikes

---

## Symptoms

- latency spikes during traffic bursts
- delayed scaling
- cost spikes due to overreaction

---

# Rightsizing EC2 and Node Groups

## Definition

Matching instance types to actual workload needs.

---

## Issues

- using oversized instances
- poor instance-family selection
- uneven AZ distribution

---

## Fix

- use mixed instance types
- optimize node shapes (CPU vs memory optimized)
- use spot instances where safe

---

# Kafka / DB Cost Amplification

## Kafka

- over-retention = storage explosion
- excessive partitions = broker overhead

---

## Databases

- over-replication = compute + storage cost
- inefficient queries = scale-up requirement

---

## Key Insight

> inefficiency in data systems multiplies cost across entire platform

---

# Cost Anomaly Detection Systems

## Definition

Systems that detect sudden cost spikes.

---

## What they monitor

- EC2 usage spikes
- storage growth
- network egress
- Kafka throughput changes

---

## Failure Modes

- alerts too noisy → ignored
- delayed detection → cost runaway

---

# Multi-Account AWS Cost Governance

## Definition

Cost control across multiple AWS accounts.

---

## Challenges

- fragmented visibility
- inconsistent tagging
- shared infrastructure costs

---

## Fix

- enforce tagging standards
- centralized billing dashboards
- service-level cost attribution

---

# Real Production Incident Patterns

---

## Pattern 1: “Cost spike after deployment”

Cause:
- increased resource requests per pod

---

## Pattern 2: “Cluster scaling but low CPU usage”

Cause:
- fragmentation / bin packing inefficiency

---

## Pattern 3: “Sudden NAT gateway bill explosion”

Cause:
- increased external traffic or retries

---

## Pattern 4: “Kafka cost doubling”

Cause:
- partition explosion or retention misconfig

---

# Debugging Flow

    Step 1: Identify cost spike domain
        ↓
    Step 2: Map to infrastructure layer (compute / storage / network)
        ↓
    Step 3: Check recent deployments or config changes
        ↓
    Step 4: Analyze utilization vs allocation
        ↓
    Step 5: Identify inefficiency source (requests, scaling, traffic)

---

# Senior SRE Mental Model

Junior:
- “We are spending too much”

Mid-level:
- “Check resource usage”

Senior:
- “Is this cost driven by inefficiency, overprovisioning, or scaling misconfiguration?”

Principal:
- “Is the system structurally inefficient in how it converts load into compute?”

---

# Interview Answer Template

When analyzing cost or capacity issues, I first identify whether the increase is driven by compute, storage, networking, or scaling inefficiencies. I then compare actual resource utilization against allocated requests to detect overprovisioning or bin packing inefficiencies in Kubernetes. I also analyze autoscaling behavior to determine whether scaling is reactive or oscillatory. On the AWS side, I examine instance types, network egress, and managed service usage to identify cost drivers. I mitigate by right-sizing workloads, improving autoscaling policies, reducing fragmentation, and enforcing resource governance policies, followed by long-term optimization through profiling, workload consolidation, and cost anomaly detection systems.

---

# Key Takeaways

- Cost is a reliability signal at scale
- Kubernetes fragmentation causes hidden inefficiency
- Autoscaling can increase cost if misconfigured
- Data systems (Kafka/DB) amplify cost rapidly
- Rightsizing is continuous, not one-time

---

# End of Module 13
