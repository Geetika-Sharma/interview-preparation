# 05 - Kubernetes + AWS Combined Failure Scenarios (Senior SRE Interview Guide)

A production-grade deep dive into real-world incidents where Kubernetes failures and AWS infrastructure failures interact to create large-scale cascading outages.

---

# Table of Contents

- Introduction
- Why Combined Failures Are Dangerous
- Core Failure Interaction Model
- Scenario 1: EC2 + Scheduling Collapse
- Scenario 2: EBS + PVC + Pod Failures
- Scenario 3: VPC + CNI Networking Breakdown
- Scenario 4: NAT + External API Outage Amplification
- Scenario 5: IAM + Cluster Bootstrap Failure
- Scenario 6: Multi-AZ Partial Outage
- Cascading Failure Chain Patterns
- Debugging Strategy
- Senior SRE Mental Model
- Interview Answer Template
- Key Takeaways

---

# Introduction

In real production systems, failures rarely stay in one layer.

Instead:

> Kubernetes + AWS failures combine and amplify each other.

This creates:
- harder debugging
- misleading symptoms
- cascading outages
- delayed recovery

---

# Why Combined Failures Are Dangerous

Because they:
- hide root cause across layers
- create false signals in Kubernetes
- overload multiple subsystems at once
- trigger retry storms and autoscaler delays simultaneously

---

# Core Failure Interaction Model

    AWS Layer Failure
          ↓
    Kubernetes Symptoms
          ↓
    Application Degradation
          ↓
    Retry / Load Amplification
          ↓
    System-Wide Outage

---

# Scenario 1: EC2 + Scheduling Collapse

## What happens

- EC2 capacity runs out in a region/AZ
- Cluster autoscaler cannot provision nodes
- Kubernetes scheduler has no valid nodes

---

## Symptoms

- Pods stuck in Pending
- Events show "Insufficient cpu"
- Cluster appears healthy but unusable

---

## Hidden AWS Signal

- EC2 "capacity-available" API shows insufficient capacity

---

## Cascading Effect

- Deployment blocked
- Traffic shifts to other services
- Increased load elsewhere → secondary failures

---

# Scenario 2: EBS + PVC + Pod Failures

## What happens

- EBS volume attach latency increases
- PVC remains in Pending state
- Pods stuck in ContainerCreating

---

## Symptoms

- Only stateful workloads affected
- No CPU/memory pressure
- Sudden storage-related timeouts

---

## Cascading Effect

- Databases slow down
- Downstream services retry
- Increased load → system instability

---

# Scenario 3: VPC + CNI Networking Breakdown

## What happens

- VPC routing issue or SG misconfiguration
- CNI plugin cannot establish pod networking

---

## Symptoms

- Random pod-to-pod failures
- DNS works but service calls fail
- Cross-node traffic broken

---

## Cascading Effect

- Partial service failure
- Retry storms amplify traffic
- Latency spikes across system

---

# Scenario 4: NAT Gateway + External API Collapse

## What happens

- NAT gateway hits bandwidth or port exhaustion
- External API calls start failing

---

## Symptoms

- External dependencies fail intermittently
- Internal services appear healthy
- Timeout errors increase

---

## Cascading Effect

- Retry storms increase outbound traffic
- NAT overload worsens
- Full external dependency outage

---

# Scenario 5: IAM + Cluster Bootstrap Failure

## What happens

- IAM role propagation delay or misconfiguration
- New nodes cannot join cluster

---

## Symptoms

- Node group stuck in provisioning
- Pods cannot schedule due to no capacity

---

## Cascading Effect

- Autoscaler loops
- Cluster capacity shrinks over time
- Deployment failures escalate

---

# Scenario 6: Multi-AZ Partial Outage

## What happens

- One AZ in region becomes degraded
- Node groups unevenly distributed

---

## Symptoms

- Only some Pods fail scheduling
- Traffic imbalance across zones
- Latency spikes in affected AZ

---

## Cascading Effect

- Load shifts to healthy AZs
- Overload in remaining zones
- Secondary degradation

---

# Cascading Failure Chain Patterns

## Pattern 1: Capacity → Retry → Amplification

AWS capacity issue → Pending pods → retries → overload

---

## Pattern 2: Storage → DB slowdown → System-wide latency

EBS issue → DB lag → service retries → full outage

---

## Pattern 3: Networking → Partial failure → retry storm

VPC issue → intermittent failures → retry amplification

---

## Pattern 4: AZ degradation → load shift → secondary overload

Single AZ failure → traffic redistribution → overload elsewhere

---

# Debugging Strategy

    Step 1: Identify symptom type
        - Scheduling issue?
        - Latency issue?
        - Connectivity issue?

    Step 2: Check Kubernetes layer first
        - kubectl describe pod
        - Events

    Step 3: Check AWS dependency layer
        - EC2 capacity
        - EBS health
        - VPC routing
        - IAM status

    Step 4: Correlate cross-layer behavior

    Step 5: Identify amplification loop

---

# Senior SRE Mental Model

Junior:
- “Pods are broken”

Mid-level:
- “Check Kubernetes events and nodes”

Senior:
- “Which AWS subsystem is constraining Kubernetes scheduling or networking?”

Principal:
- “Where is the systemic dependency chain causing multi-layer amplification failure?”

---

# Interview Answer Template

When diagnosing combined Kubernetes and AWS failures, I first determine whether the primary symptom is scheduling, networking, storage, or latency-related. I then inspect Kubernetes-level signals such as pod events and node status to identify immediate constraints. After that, I map the issue to underlying AWS dependencies such as EC2 capacity, EBS volume health, VPC networking, or IAM configuration. I also check for cascading effects like retry storms or autoscaler delays that amplify the failure. Once the root cause layer is identified, I apply immediate mitigation such as scaling resources or isolating faulty zones, and follow up with long-term improvements like multi-AZ resilience, better retry controls, and reducing tight coupling between services and AWS dependencies.

---

# Key Takeaways

- Kubernetes failures often originate in AWS layers
- Combined failures are multi-layer cascades
- Retry storms amplify AWS issues into system outages
- Multi-AZ imbalance is a common hidden failure source
- Senior engineers think in dependency chains, not isolated systems

---

# End of Module 05
