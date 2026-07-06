# 02 - Kubernetes Networking (Senior SRE Interview Guide)

A production-grade deep dive into Kubernetes networking failures, debugging, and real-world incident patterns at scale (EKS, multi-AZ, high-throughput microservices, 70B+ edge requests/day systems).

---

# Table of Contents

- Introduction
- Why Kubernetes Networking Fails at Scale
- Core Networking Model
- Key Components
- DNS & Service Discovery
- CNI Networking Model
- Cluster-to-Cluster Communication
- AWS Networking Layer (VPC Context)
- Common Failure Modes (15+)
- Real Production Incident Patterns
- Debugging Flow
- Senior SRE Mental Model
- Interview Answer Template
- Key Takeaways

---

# Introduction

Kubernetes networking is the layer that connects everything:

- Pod ↔ Pod communication
- Service discovery (DNS)
- Ingress / Load balancer traffic
- Cross-node communication
- External AWS network connectivity

At scale, networking issues are:
- intermittent
- non-deterministic
- cross-layer (K8s + Linux + AWS)

---

# Why Kubernetes Networking Fails at Scale

Networking breaks due to:

- DNS overload (CoreDNS saturation)
- CNI plugin instability (Calico / Cilium issues)
- Node-level routing issues
- AWS VPC limits / misconfigurations
- Security group misalignment
- iptables / eBPF rule exhaustion
- Cross-zone latency spikes

Most incidents are **not application bugs**.

They are **network path failures**.

---

# Core Networking Model

Every Pod gets:

- IP address
- Routing rules
- DNS resolution via CoreDNS
- Virtual network interface (veth pair)

---

# Key Flow

    Pod A
      |
      v
    veth pair
      |
      v
    CNI plugin (Calico / Cilium)
      |
      v
    Node network stack (iptables / eBPF)
      |
      v
    VPC routing (AWS)
      |
      v
    Pod B / Service / External API

---

# Core Components

## CoreDNS
Handles DNS resolution inside cluster.

## CNI Plugin
Manages Pod networking (Calico, Cilium, Weave).

## kube-proxy
Manages Service routing via iptables or IPVS.

## VPC (AWS)
Provides underlying network routing between nodes.

---

# DNS & Service Discovery Failures

## What happens

Pods cannot resolve services like:

    my-service.default.svc.cluster.local

---

## Symptoms

- High latency or timeout in service calls
- "Temporary failure in name resolution"
- Intermittent connectivity failures

---

## Root Causes

### 1. CoreDNS overload
- Too many DNS queries
- CPU saturation in CoreDNS pods

### 2. Node DNS cache issues
- Stale or missing DNS entries

### 3. Network policy blocking DNS
- UDP/TCP 53 blocked

### 4. Upstream latency (VPC resolver issues)

---

## Fix

- Scale CoreDNS replicas
- Enable caching
- Optimize DNS queries in apps
- Check NetworkPolicies

---

# CNI Networking Failures

## Definition

CNI (Container Network Interface) is responsible for Pod IP networking.

---

## Symptoms

- Pods cannot communicate across nodes
- Random packet drops
- Intermittent connectivity loss

---

## Root Causes

### 1. iptables rule overflow
- Too many services

### 2. eBPF map exhaustion (Cilium)
- Kernel limits reached

### 3. Broken node networking agent
- CNI daemon crash

### 4. MTU mismatch
- Packet fragmentation issues

---

## Fix

- Restart CNI pods
- Tune MTU settings
- Reduce service complexity
- Upgrade CNI version

---

# kube-proxy Failures

## Definition

kube-proxy handles Service routing.

---

## Modes
- iptables mode
- IPVS mode

---

## Failures

### 1. iptables rule explosion
- Too many services/endpoints

### 2. Stale endpoint updates
- Services not updating correctly

### 3. kube-proxy crash loop
- Node routing broken

---

## Fix

- Switch to IPVS or eBPF
- Reduce service churn
- Restart kube-proxy

---

# AWS VPC Networking Issues

## Definition

Underlying cloud networking layer in EKS.

---

## Failures

### 1. Security group misconfiguration
- Traffic blocked between nodes

### 2. ENI limits reached
- Cannot attach new network interfaces

### 3. Route table misconfiguration
- Cross-AZ traffic broken

### 4. NAT gateway saturation
- External API failures

---

## Symptoms

- Cross-node communication fails
- External APIs time out
- Partial zone outages

---

## Fix

- Increase ENI limits
- Fix security groups
- Optimize NAT gateway scaling

---

# 15+ Common Networking Failure Modes

## 1. CoreDNS saturation
## 2. DNS cache poisoning/staleness
## 3. CNI plugin crash
## 4. iptables rule overflow
## 5. eBPF map exhaustion
## 6. kube-proxy failure
## 7. MTU mismatch
## 8. Security group blocking
## 9. ENI limit exhaustion
## 10. NAT gateway saturation
## 11. Cross-AZ latency spikes
## 12. Packet fragmentation
## 13. NetworkPolicy misconfiguration
## 14. Service endpoint staleness
## 15. Node network interface failure

---

# Real Production Incident Patterns

## Pattern 1: “Everything is slow”

Cause:
- DNS latency or CoreDNS overload

---

## Pattern 2: “Only cross-zone traffic is broken”

Cause:
- VPC routing or security group issue

---

## Pattern 3: “Random service failures”

Cause:
- CNI instability or packet drops

---

## Pattern 4: “External APIs failing”

Cause:
- NAT gateway saturation

---

# Debugging Flow

    Networking Issue Detected
            |
            v
    Check DNS first (CoreDNS)
            |
            v
    Check Service routing (kube-proxy)
            |
            v
    Check Pod-to-Pod connectivity
            |
            v
    Check CNI plugin status
            |
            v
    Check Node networking
            |
            v
    Check AWS VPC layer
            |
            v
    Identify broken layer

---

# Senior SRE Mental Model

Junior:
- “Service is down”

Mid-level:
- “Check logs and restart pods”

Senior:
- “Which network layer is failing: DNS, CNI, service routing, or VPC?”

Principal:
- “Is this a distributed systems networking failure or infrastructure routing degradation?”

---

# Interview Answer Template

When diagnosing Kubernetes networking issues, I first determine whether the failure is DNS, service routing, pod-to-pod connectivity, or external network communication. I start by validating CoreDNS health for DNS resolution issues, then check kube-proxy and service endpoints for routing problems. If cross-pod communication is affected, I inspect the CNI plugin and node networking stack for packet loss or rule exhaustion. Finally, I validate AWS VPC components such as security groups, route tables, and NAT gateways. Once the layer is identified, I apply targeted mitigation and then implement systemic fixes such as scaling CoreDNS, simplifying network policies, or improving CNI stability.

---

# Key Takeaways

- Kubernetes networking is multi-layered (DNS → CNI → Service → VPC)
- Most failures are cross-layer, not single component issues
- DNS (CoreDNS) is the most common bottleneck
- CNI issues cause the hardest-to-debug outages
- AWS VPC misconfigurations often mimic K8s failures

---

# Next Section

## 03 - Cross-Service Cascading Failures

We will cover:
- Distributed failure propagation
- Retry storms
- Circuit breaker failures
- Kafka / queue backpressure collapse
- Real production incident chains at scale
