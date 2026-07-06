# 12 - Service Mesh + Service Discovery Failure Systems (Senior SRE Interview Guide)

A production-grade deep dive into service-to-service communication failures using service mesh systems (Istio, Envoy) and service discovery systems (Consul, Kubernetes DNS) at large scale.

---

# Table of Contents

- Introduction
- Why Service Mesh Exists
- Service Discovery vs Service Mesh
- Core Components (Envoy, Control Plane, DNS)
- Service Discovery Failures
- DNS-Based Failure Modes (CoreDNS)
- Service Mesh Failure Modes (Istio / Envoy)
- mTLS & Identity Failures
- Traffic Routing Misconfigurations
- Sidecar Injection & Lifecycle Issues
- Partial Mesh Degradation Scenarios
- Real Production Incident Patterns
- Debugging Flow
- Senior SRE Mental Model
- Interview Answer Template
- Key Takeaways

---

# Introduction

At scale, services do NOT talk directly to each other reliably.

They rely on:
- service discovery (where is service?)
- service mesh (how to route traffic safely?)

So when this layer breaks:

> Everything appears “randomly broken”

---

# Why Service Mesh Exists

Service mesh solves:
- traffic routing
- retries / timeouts
- mTLS encryption
- observability
- circuit breaking

But it also introduces:

> another complex failure domain

---

# Service Discovery vs Service Mesh

## Service Discovery

> “How do I find another service?”

Examples:
- Kubernetes DNS
- Consul

---

## Service Mesh

> “Once I find it, how do I safely communicate?”

Examples:
- Istio
- Linkerd
- Envoy

---

# Core Components

## Envoy Proxy

- sidecar proxy per pod
- handles all traffic in/out

---

## Control Plane

- configures Envoy behavior
- pushes routing rules

---

## DNS (CoreDNS)

- resolves service names → IPs

---

# 1. Service Discovery Failures

## Definition

Services cannot locate other services.

---

## Failure Modes

### 1. DNS resolution failure
- CoreDNS overloaded or down

### 2. Stale service records
- outdated endpoints

### 3. Incorrect service registration

---

## Symptoms

- intermittent “service not found”
- request timeouts
- sudden spikes in 5xx errors

---

## Fix

- restart CoreDNS
- validate service registry
- increase DNS cache TTL carefully

---

# 2. CoreDNS Failures

## Definition

DNS layer inside Kubernetes becomes overloaded.

---

## Failure Modes

### 1. High QPS from services
### 2. Cache misses
### 3. CPU saturation

---

## Symptoms

- global latency spikes
- random service failures
- API calls fail intermittently

---

## Fix

- scale CoreDNS replicas
- enable caching
- reduce DNS query rate

---

# 3. Service Mesh (Envoy / Istio) Failures

## Definition

Traffic routing layer between services breaks.

---

## Failure Modes

### 1. Sidecar misconfiguration
- wrong routing rules

### 2. Control plane failure
- config not pushed to proxies

### 3. Envoy overload
- CPU/memory exhaustion in sidecars

---

## Symptoms

- partial service outage
- inconsistent routing behavior
- unexpected 503 errors

---

## Fix

- restart sidecars
- rollback mesh config
- isolate faulty routing rules

---

# 4. mTLS & Identity Failures

## Definition

Mutual TLS authentication between services fails.

---

## Failure Modes

### 1. Certificate expiration
### 2. Clock skew between services
### 3. Identity mismatch

---

## Symptoms

- sudden service-to-service auth failures
- 403 / 401 errors at scale
- healthy pods but broken communication

---

## Fix

- rotate certificates
- fix time synchronization (NTP)
- validate service identities

---

# 5. Traffic Routing Misconfigurations

## Definition

Incorrect routing rules in mesh cause traffic misdirection.

---

## Failure Modes

### 1. Canary routing broken
### 2. Weighted routing misconfigured
### 3. Cross-service routing loops

---

## Symptoms

- unexpected traffic shifts
- version skew in production
- latency spikes in specific services

---

## Fix

- rollback routing config
- verify traffic policies
- enforce config validation pipelines

---

# 6. Sidecar Injection & Lifecycle Issues

## Definition

Envoy sidecars are not correctly injected or updated.

---

## Failure Modes

### 1. Missing sidecars in pods
### 2. Version mismatch between sidecars
### 3. Crash loops in proxy containers

---

## Symptoms

- some pods bypass mesh
- inconsistent observability
- random traffic failures

---

## Fix

- re-inject sidecars
- enforce admission controllers
- align proxy versions

---

# 7. Partial Mesh Degradation

## Definition

Only part of the service mesh is broken.

---

## Causes

- control plane partial outage
- regional config drift
- version skew in proxies

---

## Symptoms

- only some services fail
- inconsistent routing
- hard-to-reproduce issues

---

# 8. Real Production Incident Patterns

---

## Pattern 1: “Everything is random 503 errors”

Cause:
- Envoy misrouting or control plane issue

---

## Pattern 2: “Some services cannot resolve others”

Cause:
- CoreDNS overload or failure

---

## Pattern 3: “Service works but auth fails”

Cause:
- mTLS certificate issue

---

## Pattern 4: “Only one region broken”

Cause:
- mesh control plane regional failure

---

# 9. Debugging Flow

    Step 1: Check DNS resolution
        ↓
    Step 2: Check CoreDNS health
        ↓
    Step 3: Check service mesh control plane
        ↓
    Step 4: Check Envoy sidecars
        ↓
    Step 5: Validate mTLS and certificates
        ↓
    Step 6: Check routing policies

---

# 10. Senior SRE Mental Model

Junior:
- “Service is broken”

Mid-level:
- “Check DNS and service endpoints”

Senior:
- “Is this a discovery issue, mesh routing issue, or identity/auth issue?”

Principal:
- “Is the communication layer itself introducing systemic uncertainty in service connectivity?”

---

# 11. Interview Answer Template

When diagnosing service mesh or service discovery issues, I first determine whether the failure originates from DNS resolution, service registry, or mesh traffic routing. I check CoreDNS health to ensure services can resolve each other correctly, then validate service mesh components such as Envoy sidecars and control plane configuration. I also inspect mTLS certificate validity and identity consistency to rule out authentication failures. If partial failures exist, I look for configuration drift or control plane degradation affecting only subsets of services. I mitigate by restoring DNS or mesh stability, rolling back routing changes, and ensuring consistent sidecar behavior across the cluster, followed by strengthening observability and configuration validation pipelines.

---

# 12. Key Takeaways

- Service discovery is the foundation of microservice communication
- Service mesh adds control but also failure complexity
- DNS failures look like random outages
- mTLS failures look like authentication issues, not infrastructure issues
- Partial mesh failures are hardest to debug in production

---

# End of Module 12
