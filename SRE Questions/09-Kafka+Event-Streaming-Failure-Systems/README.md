# 09 - Kafka + Event Streaming Failure Systems (Senior SRE Interview Guide)

A production-grade deep dive into large-scale Kafka and event-driven system failures in high-throughput environments (hundreds of services, billions of messages/day, real-time pipelines).

---

# Table of Contents

- Introduction
- Why Kafka Systems Fail at Scale
- Kafka Architecture Refresher
- Producer Failures
- Broker Failures
- Consumer Failures
- Consumer Group Rebalance Storms
- Partition Imbalance Problems
- Backpressure Collapse
- Exactly-Once Semantics Failures
- Real Production Incident Patterns
- Debugging Flow
- Senior SRE Mental Model
- Interview Answer Template
- Key Takeaways

---

# Introduction

Kafka is not just a messaging system.

At scale, it becomes:

> The nervous system of the entire infrastructure.

So when Kafka degrades:
- microservices stall
- data pipelines lag
- caches become stale
- APIs return inconsistent results

---

# Why Kafka Systems Fail at Scale

Kafka failures happen due to:

- uneven partition distribution
- consumer lag explosion
- broker disk/IO bottlenecks
- network saturation
- misconfigured retention policies
- uncontrolled retries from producers/consumers

---

# Kafka Architecture Refresher

Kafka consists of:

- **Producers** → send messages
- **Brokers** → store messages
- **Partitions** → parallelism unit
- **Consumers** → process messages
- **Consumer Groups** → coordination layer

---

# 1. Producer Failures

## Definition

Producers are services writing data into Kafka.

---

## Failure Modes

### 1. Retry storm from producer
- broker slow → producer retries aggressively

### 2. Batch size misconfiguration
- too large → memory pressure
- too small → throughput loss

### 3. ACK misconfiguration
- ack=all → high latency
- ack=1 → risk of data loss

---

## Symptoms

- increased write latency
- message duplication
- throughput drop

---

## Fix

- tune retries with backoff
- adjust batching strategy
- optimize ACK settings

---

# 2. Broker Failures

## Definition

Kafka brokers store and replicate data.

---

## Failure Modes

### 1. Disk IO saturation
- log segments too large

### 2. Broker crash / restart loops

### 3. Network partition between brokers

### 4. Under-replicated partitions (URP)

---

## Symptoms

- increased produce/consume latency
- ISR shrinkage
- data replication lag

---

## Fix

- increase disk throughput
- rebalance partitions
- scale broker cluster

---

# 3. Consumer Failures

## Definition

Consumers process messages from Kafka topics.

---

## Failure Modes

### 1. Slow consumer processing
- downstream DB slow
- heavy computation per message

### 2. Consumer crash loops

### 3. Offset commit delays

---

## Symptoms

- increasing consumer lag
- stale downstream systems
- delayed pipelines

---

## Fix

- scale consumer instances
- optimize processing logic
- decouple heavy workloads

---

# 4. Consumer Group Rebalance Storms

## Definition

Frequent reassignments of partitions across consumers.

---

## Why it happens

- consumer crashes
- scaling events
- network instability
- heartbeat timeout

---

## Symptoms

- unstable lag
- repeated rebalancing logs
- throughput collapse

---

## Fix

- use static membership
- increase session timeout
- stabilize consumer scaling

---

# 5. Partition Imbalance Problem

## Definition

Some partitions receive much more traffic than others.

---

## Why it happens

- bad partition key design
- skewed traffic patterns

---

## Symptoms

- hot partitions
- uneven consumer load
- lag concentrated in few partitions

---

## Fix

- improve partition key strategy
- increase partition count
- redistribute traffic

---

# 6. Backpressure Collapse

## Definition

Kafka cannot absorb incoming load fast enough, causing system-wide slowdown.

---

## Failure Chain

    Producers → Kafka overload → Consumer lag → retries → more load

---

## Symptoms

- increasing lag across all topics
- system-wide slowdown
- cascading retries in microservices

---

## Fix

- throttle producers
- scale brokers
- enforce backpressure at API layer

---

# 7. Exactly-Once Semantics Failures (EOS)

## Definition

Kafka guarantees exactly-once processing under strict conditions.

---

## Failure Modes

- duplicate processing
- offset commit mismatch
- transactional producer failure

---

## Symptoms

- inconsistent downstream state
- duplicate events in DB
- data integrity issues

---

## Fix

- enforce idempotency
- verify transaction boundaries
- use deduplication layers

---

# 8. Real Production Incident Patterns

---

## Pattern 1: “Kafka lag explosion”

Cause:
- consumer slowdown + producer continues full load

---

## Pattern 2: “Everything is delayed”

Cause:
- broker disk IO saturation

---

## Pattern 3: “Random service inconsistency”

Cause:
- partition skew + lag mismatch

---

## Pattern 4: “System-wide retry storm”

Cause:
- Kafka slowdown triggers upstream retries

---

# 9. Debugging Flow

    Step 1: Check consumer lag metrics
        ↓
    Step 2: Check broker health (disk, CPU, ISR)
        ↓
    Step 3: Check producer throughput
        ↓
    Step 4: Check partition distribution
        ↓
    Step 5: Identify bottleneck layer

---

# 10. Senior SRE Mental Model

Junior:
- “Kafka is slow”

Mid-level:
- “Check consumers and brokers”

Senior:
- “Is this a partition imbalance, broker saturation, or consumer backpressure issue?”

Principal:
- “Is Kafka acting as a buffer or becoming a system-wide failure amplifier?”

---

# 11. Interview Answer Template

When diagnosing Kafka-related failures, I first determine whether the issue is on the producer, broker, or consumer side. I check consumer lag to understand processing delays, then validate broker health including disk usage, ISR status, and network latency. I also inspect partition distribution to identify skew or hot partitions. If lag is increasing system-wide, I look for backpressure collapse where slow consumption causes upstream retries and system amplification. I mitigate by scaling consumers, throttling producers, and stabilizing broker performance, followed by long-term fixes like improving partition strategy, tuning retention policies, and ensuring backpressure mechanisms exist across the system.

---

# 12. Key Takeaways

- Kafka is a distributed buffer, not just a queue
- Partition design determines system scalability
- Consumer lag is the most important signal
- Backpressure failure leads to system-wide outages
- Kafka issues often amplify into microservice failures

---

# End of Module 09
