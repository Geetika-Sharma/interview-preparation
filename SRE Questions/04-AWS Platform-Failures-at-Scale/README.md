# 04 - AWS Platform Failures at Scale (Senior SRE Interview Guide)

A production-focused deep dive into AWS infrastructure failures that impact Kubernetes clusters, microservices, and large-scale distributed systems (EKS, EC2 fleets, multi-account setups).

---

# Table of Contents

- Introduction
- Why AWS Failures Matter in Kubernetes Systems
- Core AWS Dependency Model
- EC2 Failures
- EBS Failures
- VPC Networking Failures
- IAM & Control Plane Issues
- Load Balancer Failures
- Region & AZ-Level Outages
- Real Production Incident Patterns
- Debugging Flow
- Senior SRE Mental Model
- Interview Answer Template
- Key Takeaways

---

# Introduction

At large scale, Kubernetes is not an isolated system.

It depends heavily on AWS:

- EC2 (compute)
- EBS (storage)
- VPC (networking)
- IAM (permissions)
- ELB/ALB/NLB (traffic routing)
- Route53 (DNS)

So when AWS degrades:

> Kubernetes failures are often just symptoms.

---

# Why AWS Failures Matter in Kubernetes Systems

Even if your application is perfect:

- Nodes may not launch
- Volumes may not attach
- Networking may break
- Pods may be stuck Pending
- Traffic routing may degrade

AWS becomes the **foundation of failure propagation**.

---

# Core AWS Dependency Model

    Kubernetes Cluster
          |
          v
    EC2 (Nodes)
          |
          v
    EBS (Storage)
          |
          v
    VPC (Networking Layer)
          |
          v
    IAM + Control Plane APIs

---

# 1. EC2 Failures (Compute Layer)

## Definition

Failures in provisioning or running virtual machines.

---

## Common Issues

### 1. Instance type capacity exhaustion
- AWS has no capacity in selected AZ

### 2. Slow instance provisioning
- Boot time delays under load

### 3. Spot instance interruptions
- Preemptions in burst traffic

### 4. IMDS (metadata service) failures
- Node bootstrap issues

---

## Symptoms

- Pods stuck Pending
- Cluster autoscaler not scaling
- Node groups stuck in provisioning

---

## Fix

- Switch instance types
- Use multiple AZs
- Use mixed instance policies

---

# 2. EBS Failures (Storage Layer)

## Definition

Issues with block storage volumes attached to EC2 instances.

---

## Common Issues

### 1. Volume attach delay
- Pod stuck waiting for storage

### 2. Volume limit per AZ
- Cannot attach new volumes

### 3. I/O throttling
- Severe performance degradation

---

## Symptoms

- PVC stuck in Pending
- Pod stuck ContainerCreating
- High disk latency

---

## Fix

- Pre-provision volumes
- Increase IOPS
- Switch storage class

---

# 3. VPC Networking Failures

## Definition

Failures in AWS virtual network layer affecting connectivity.

---

## Common Issues

### 1. Security group misconfiguration
- Traffic blocked between nodes

### 2. Route table misalignment
- Cross-subnet communication failure

### 3. ENI exhaustion
- Cannot attach network interfaces

### 4. NAT gateway saturation
- External API failures

---

## Symptoms

- Cross-node communication failure
- External API timeouts
- DNS resolution inconsistencies

---

## Fix

- Increase ENI limits
- Fix security groups
- Add NAT gateways per AZ

---

# 4. IAM & Control Plane Issues

## Definition

Failures due to AWS permissions or API control delays.

---

## Common Issues

### 1. IAM propagation delay
- Newly updated roles not effective immediately

### 2. EKS API throttling
- Cluster control plane slow

### 3. STS token expiration issues

---

## Symptoms

- Node bootstrap failures
- kubectl command failures
- Deployment stuck in progress

---

## Fix

- Retry with exponential backoff
- Validate IAM policies
- Reduce API call rate

---

# 5. Load Balancer Failures (ALB/NLB)

## Definition

Traffic routing failures at AWS load balancing layer.

---

## Common Issues

### 1. Target group unhealthy
### 2. Slow health checks
### 3. Cross-zone load imbalance
### 4. DNS propagation delay

---

## Symptoms

- 5xx errors
- Partial service outage
- Uneven traffic distribution

---

## Fix

- Adjust health check thresholds
- Re-register targets
- Optimize routing policies

---

# 6. Region & AZ-Level Failures

## Definition

Large-scale AWS infrastructure degradation in a region or availability zone.

---

## Types

### 1. Single AZ outage
- Node groups in one AZ fail

### 2. Regional degradation
- API latency spikes
- Multi-service impact

---

## Symptoms

- Widespread Pod failures
- Autoscaler ineffective
- Multiple services degraded

---

## Fix

- Shift traffic across AZs
- Failover to backup region
- Reduce dependency on single AZ

---

# Real Production Incident Patterns

## Pattern 1: “Pods stuck Pending globally”

Cause:
- EC2 capacity exhaustion in region

---

## Pattern 2: “Storage-related outage”

Cause:
- EBS volume attach delays

---

## Pattern 3: “Random network failures”

Cause:
- VPC security group or NAT issues

---

## Pattern 4: “Everything is slow”

Cause:
- Regional AWS API degradation

---

# Debugging Flow

    Issue Detected
          |
          v
    Check Kubernetes layer first
          |
          v
    Check node provisioning (EC2)
          |
          v
    Check storage (EBS / PVC)
          |
          v
    Check networking (VPC / SG / NAT)
          |
          v
    Check IAM / API control plane
          |
          v
    Map failure to AWS dependency layer

---

# Senior SRE Mental Model

Junior:
- “Kubernetes is broken”

Mid-level:
- “Check nodes and pods”

Senior:
- “Which AWS dependency layer is failing: compute, storage, networking, or control plane?”

Principal:
- “Is this a regional infrastructure degradation or a system design over-dependence on a single AWS subsystem?”

---

# Interview Answer Template

When diagnosing AWS-related failures in a Kubernetes environment, I first determine whether the issue originates from compute, storage, networking, or control plane layers. I start by checking node provisioning status to identify EC2 capacity or scaling issues. Then I validate storage health through EBS and PVC states. Next, I inspect VPC networking components such as security groups, NAT gateways, and routing tables for connectivity issues. Finally, I check IAM permissions and control plane API health for authorization or throttling problems. Once the failing layer is identified, I apply immediate mitigation such as scaling resources or adjusting configurations, followed by long-term improvements like multi-AZ resilience, capacity planning, and reducing single points of dependency on AWS services.

---

# Key Takeaways

- AWS is a foundational dependency layer for Kubernetes
- Most “Kubernetes issues” are actually AWS issues
- Failures can occur at compute, storage, networking, or control plane level
- Multi-AZ design is critical for resilience
- Senior engineers think in dependency layers, not symptoms

---

# End of Module 04
