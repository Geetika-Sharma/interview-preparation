# 03 - Cross-Service Cascading Failures (Senior SRE Interview Guide)

A production-grade deep dive into distributed failure propagation, retry storms, and system-wide outages in large microservice architectures (Kubernetes + Kafka + AWS scale systems).

---

# Table of Contents

- Introduction
- Why Cascading Failures Happen
- Core Concepts
- Failure Propagation Model
- Retry Storms
- Circuit Breakers
- Backpressure Collapse
- Kafka / Queue Failures
- Real Production Incident Patterns
- Debugging Flow
- Senior SRE Mental Model
- Interview Answer Template
- Key Takeaways

---

# Introduction

A cross-service cascading failure is when:

> A small failure in one service spreads across multiple services and becomes a system-wide outage.

At scale, this is one of the most dangerous failure modes because:
- It starts small
- It amplifies silently
- It overwhelms the entire system

---

# Why Cascading Failures Happen

Cascading failures occur due to **feedback loops** in distributed systems:

- Service A fails → Service B retries
- Service B retries → increases load on Service A
- Service A becomes slower → more retries happen
- System spirals into collapse

---

# Core Concepts

## Downstream Dependency
A service that another service relies on.

## Upstream Service
A service that calls downstream services.

## Backpressure
A mechanism where a system signals it cannot handle more load.

## Retry Policy
Logic that retries failed requests automatically.

## Circuit Breaker
Stops traffic to a failing service to prevent overload.

---

# Failure Propagation Model

    Service A
        |
        v
    Service B
        |
        v
    Service C (DB / Kafka / external API)

If Service C slows down:

    C slows → B retries → A retries → system overload

---

# Retry Storms (Critical Concept)

## Definition

A retry storm happens when multiple services retry failed requests simultaneously, increasing load instead of reducing it.

---

## Why it happens

- No retry limits
- No jitter in retries
- No circuit breaker
- Aggressive client retry logic

---

## Symptoms

- CPU spikes across multiple services
- Sudden latency explosion
- Increasing request volume during failure

---

## Real pattern

    Failure → Retry → More load → More failure → Infinite loop

---

## Fix

- Add exponential backoff
- Add jitter
- Limit retry attempts
- Implement circuit breakers

---

# Circuit Breakers

## Definition

A circuit breaker stops traffic to a failing service after repeated failures.

---

## States

- CLOSED → normal operation
- OPEN → block requests
- HALF-OPEN → test recovery

---

## Why it matters

Prevents:
- retry storms
- cascading overload
- dependency collapse

---

# Backpressure Collapse

## Definition

When a system cannot signal overload fast enough, causing uncontrolled traffic buildup.

---

## Example

- Kafka consumer lag increases
- Producers continue sending data
- Queue fills up
- Memory/disk pressure increases
- System crashes

---

## Fix

- Enforce rate limiting
- Add queue size controls
- Apply consumer throttling
- Enable load shedding

---

# Kafka / Queue Failures

## Common failure patterns

### 1. Consumer lag explosion
- Consumers too slow
- Messages accumulate

### 2. Partition imbalance
- Uneven load distribution

### 3. Broker overload
- Disk IO saturation

### 4. Rebalance storms
- Frequent consumer group rebalances

---

## Symptoms

- Delayed processing
- High latency pipelines
- Event backlog growth

---

## Fix

- Scale consumers
- Increase partitions
- Optimize processing logic

---

# Real Production Incident Patterns

## Pattern 1: “One dependency slows everything”

Cause:
- DB latency increase
- All services retry DB calls

---

## Pattern 2: “Retry storm outage”

Cause:
- No circuit breaker
- exponential retry amplification

---

## Pattern 3: “Kafka lag explosion”

Cause:
- Consumer failure + continued production

---

## Pattern 4: “Partial outage becomes full outage”

Cause:
- cascading dependency chain

---

# Debugging Flow

    System Degraded
          |
          v
    Identify first slow service
          |
          v
    Trace upstream calls
          |
          v
    Check retry rate increase
          |
          v
    Check queue/backlog systems (Kafka, SQS)
          |
          v
    Identify amplification loop
          |
          v
    Stop traffic / apply circuit breaker

---

# Key Amplification Mechanisms

## 1. Retries without limits
## 2. Lack of backpressure
## 3. Synchronous dependency chains
## 4. High fan-out architectures
## 5. No rate limiting

---

# Senior SRE Mental Model

Junior:
- “Service is slow”

Mid-level:
- “Check DB or cache latency”

Senior:
- “Is this a feedback loop caused by retries amplifying a downstream degradation?”

Principal:
- “Where is the amplification boundary missing in the system design?”

---

# Interview Answer Template

When diagnosing cross-service cascading failures, I first identify the initial point of degradation by analyzing latency and error rate spikes across services. I then trace upstream dependencies to determine how failures are propagating. I specifically check for retry amplification, queue backlogs, and circuit breaker states. If retries are uncontrolled, they often amplify downstream load, creating a feedback loop that worsens the outage. I mitigate by isolating the failing dependency, enabling circuit breakers, applying rate limits, and stopping retry storms. Finally, I implement long-term fixes such as introducing backpressure mechanisms, improving retry policies with exponential backoff and jitter, and decoupling synchronous dependencies.

---

# Key Takeaways

- Cascading failures are feedback loops, not single-point failures
- Retries are a major cause of system-wide outages
- Circuit breakers are essential for stability
- Backpressure must exist at every layer
- Most outages are amplification problems

---

# End of Module 03
