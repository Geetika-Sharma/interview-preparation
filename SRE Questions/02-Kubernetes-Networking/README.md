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

 # 02 - Kubernetes Networking (Part 2)

## Deep Debugging, Edge Cases, and Real Production Incidents

---

# Objective

This section goes beyond basic networking failures and focuses on:

- Hard-to-debug intermittent issues
- Cross-layer networking breakdowns
- Production-scale incident patterns
- Senior SRE-level diagnosis under ambiguity

---

# 1. Intermittent Connectivity Failures

## Definition

Requests sometimes succeed and sometimes fail between services.

---

## Why this happens

Intermittent failures usually indicate **partial network degradation**, not full outage.

Common causes:

- Packet drops at CNI layer
- Node-specific network corruption
- DNS cache inconsistency
- Load balancer uneven routing
- TCP connection reuse issues

---

## Symptoms

- 1–5% request failure rate
- No consistent pattern
- Logs show mixed success/failure

---

## Debug Approach

Check in this order:

    1. CoreDNS logs
    2. Node-level packet drops
    3. CNI plugin health
    4. kube-proxy endpoint consistency

---

## Fix

- Restart affected CNI pods on bad nodes
- Replace unhealthy nodes
- Enable better observability (eBPF tracing if available)

---

# 2. DNS Intermittency (Most Common Hidden Issue)

## Definition

DNS works but occasionally fails or is slow.

---

## Root Causes

### 1. CoreDNS CPU throttling
- HPA not configured correctly

### 2. Connection saturation
- Too many concurrent queries

### 3. Upstream resolver latency (VPC DNS)

### 4. Node-level DNS cache corruption

---

## Symptoms

- Sporadic "timeout resolving service"
- Latency spikes in service calls
- Higher failure rate during traffic bursts

---

## Fix

- Scale CoreDNS horizontally
- Enable NodeLocal DNSCache
- Optimize application retry behavior

---

# 3. Cross-Zone Latency Degradation

## Definition

Traffic between zones becomes slow but not fully broken.

---

## Root Causes

- AWS inter-AZ bandwidth saturation
- Asymmetric routing paths
- Load balancer uneven distribution
- VPC peering latency spikes

---

## Symptoms

- Requests succeed but slow only across zones
- Intra-zone traffic is healthy
- Elevated p99 latency

---

## Fix

- Enforce zone-aware routing
- Reduce cross-zone dependencies
- Tune load balancer distribution policies

---

# 4. Service Discovery Delay (Endpoints Staleness)

## Definition

Services take time to discover new Pods or remove old ones.

---

## Why it happens

- kube-proxy delay in syncing endpoints
- Controller lag in endpoint updates
- Large-scale endpoint churn

---

## Symptoms

- Requests hit terminating Pods
- New Pods not receiving traffic immediately
- Uneven traffic distribution

---

## Fix

- Reduce endpoint churn rate
- Optimize readiness probes
- Switch to IPVS or eBPF-based routing

---

# 5. Packet Loss at Scale

## Definition

Packets are dropped intermittently in cluster networking layer.

---

## Root Causes

### 1. Node network interface saturation
### 2. Kernel buffer exhaustion
### 3. CNI misconfiguration
### 4. MTU mismatch causing fragmentation loss

---

## Symptoms

- TCP retransmissions increase
- gRPC timeouts
- Intermittent API failures

---

## Fix

- Increase node networking limits
- Align MTU across cluster
- Upgrade CNI plugin version

---

# 6. NAT Gateway Bottleneck (External API Failures)

## Definition

Pods cannot reliably reach external services.

---

## Root Causes

- NAT gateway bandwidth saturation
- Port exhaustion (SNAT limits)
- Too many outbound connections

---

## Symptoms

- External APIs timeout
- Internal services unaffected
- Increased connection failures

---

## Fix

- Add multiple NAT gateways
- Use NAT scaling strategies
- Reduce connection churn (keep-alives)

---

# 7. CNI Control Plane Split Brain

## Definition

Different nodes have inconsistent network state.

---

## Why it happens

- CNI agent restart failures
- Partial configuration rollout
- Node reboot during updates

---

## Symptoms

- Some Pods reachable, others not
- Random cross-node failures
- Inconsistent routing behavior

---

## Fix

- Restart CNI daemonset
- Drain and replace affected nodes
- Ensure rollout consistency

---

# 8. kube-proxy Endpoint Drift

## Definition

kube-proxy routing table is outdated.

---

## Root Causes

- Endpoint update delays
- Large service churn
- kube-proxy crash loops

---

## Symptoms

- Requests go to terminated Pods
- Uneven traffic distribution
- Random 5xx errors

---

## Fix

- Restart kube-proxy
- Switch to IPVS or eBPF
- Reduce service churn

---

# 9. Hidden Network Policy Misconfiguration

## Definition

NetworkPolicies silently block traffic.

---

## Why it happens

- Overly strict ingress/egress rules
- Missing namespace selectors
- Policy conflicts

---

## Symptoms

- Only some services can communicate
- No infrastructure errors
- Appears as application bug

---

## Fix

- Audit all NetworkPolicies
- Use allow-list instead of deny-by-default incorrectly
- Test policy changes incrementally

---

# 10. Multi-Layer Failure Chain (Real Incident Pattern)

## Example

1. DNS latency increases
2. Service retries increase
3. NAT gateway saturates
4. Packet loss increases
5. App timeouts spike
6. Retry storms amplify load

---

## Result

A small network issue becomes a **full system outage**

---

# Debugging Strategy (Senior Level)

    Step 1: Identify symptom type
        - DNS issue?
        - Latency issue?
        - Connectivity issue?

    Step 2: Identify layer
        - DNS (CoreDNS)
        - Service routing (kube-proxy)
        - Pod networking (CNI)
        - Node networking
        - VPC layer (AWS)

    Step 3: Isolate scope
        - Single node?
        - Single AZ?
        - Entire cluster?

    Step 4: Validate metrics + logs

---

# Senior SRE Mental Model

Junior:
- “Network is broken”

Mid-level:
- “Check DNS and restart services”

Senior:
- “Which layer in the network stack is degrading: DNS, service routing, pod networking, or VPC?”

Principal:
- “Is this a systemic network saturation issue or a cascading failure across multiple layers?”

---

# Interview Answer Template

When diagnosing Kubernetes networking issues, I first classify whether the issue is DNS resolution, service routing, pod-to-pod communication, or external connectivity. I start by checking CoreDNS for DNS issues, followed by kube-proxy and endpoint consistency for service routing problems. If pod-level communication is affected, I inspect the CNI layer for packet loss, MTU mismatches, or agent failures. Finally, I validate AWS VPC components such as NAT gateways, security groups, and route tables. Once the failing layer is identified, I apply targeted remediation and then focus on systemic fixes such as scaling CoreDNS, simplifying network policies, or improving CNI stability.

---

# Key Takeaways

- Most networking issues are intermittent, not total failures
- DNS is the most common bottleneck
- CNI issues are the hardest to debug
- AWS networking often mimics Kubernetes failures
- Failures are usually multi-layered, not single-component

---
