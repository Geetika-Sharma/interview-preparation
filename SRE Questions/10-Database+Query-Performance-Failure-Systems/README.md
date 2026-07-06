# 10 - Database + Query Performance Failure Systems (Senior SRE Interview Guide)

A production-grade deep dive into database failures, query degradation, and large-scale data system outages in high-throughput environments (200k+ QPS systems, microservices-heavy architectures, multi-region deployments).

---

# Table of Contents

- Introduction
- Why Databases Become System Bottlenecks
- Database Architecture Refresher
- Connection Pool Exhaustion
- Slow Query & Query Plan Regression
- Lock Contention & Deadlocks
- Index Degradation Problems
- Replication Lag Failures
- Failover & Split-Brain Scenarios
- Connection Storm Cascades
- Real Production Incident Patterns
- Debugging Flow
- Senior SRE Mental Model
- Interview Answer Template
- Key Takeaways

---

# Introduction

Databases are the **strongest coupling point** in most distributed systems.

When a database degrades:

> Everything above it collapses.

At scale, most “service outages” are actually:
- DB latency issues
- DB connection saturation
- Query inefficiency amplification

---

# Why Databases Become System Bottlenecks

Databases fail under:

- high concurrent connections
- inefficient queries
- lock contention
- replication delays
- burst traffic patterns
- connection storms from retries

---

# Database Architecture Refresher

Typical production DB stack:

- Application layer (services)
- Connection pool
- Primary DB node
- Read replicas
- Storage layer (disk / SSD / network storage)

---

# 1. Connection Pool Exhaustion

## Definition

All database connections are consumed, blocking new requests.

---

## Why it happens

- too many concurrent requests
- slow queries holding connections too long
- retry storms multiplying traffic

---

## Symptoms

- requests stuck waiting for DB connection
- latency spikes across services
- timeouts at application layer

---

## Fix

- increase pool size (carefully)
- reduce query duration
- implement request throttling
- introduce caching layer

---

# 2. Slow Query & Query Plan Regression

## Definition

A query that used to be fast becomes slow due to plan changes.

---

## Why it happens

- missing indexes
- stale statistics
- schema changes
- data growth changes execution plan

---

## Symptoms

- sudden latency spike
- single endpoint causing DB load spike
- CPU spike on DB node

---

## Fix

- analyze query execution plan
- add or fix indexes
- update statistics
- rollback schema changes if needed

---

# 3. Lock Contention & Deadlocks

## Definition

Multiple transactions blocking each other.

---

## Why it happens

- high write concurrency
- poor transaction ordering
- long-running transactions

---

## Symptoms

- request pile-up
- DB CPU low but latency high
- deadlock errors

---

## Fix

- reduce transaction scope
- enforce consistent locking order
- break large transactions

---

# 4. Index Degradation Problems

## Definition

Indexes become inefficient or unused.

---

## Why it happens

- large table growth
- poor index design
- query pattern changes

---

## Symptoms

- full table scans
- high IO usage
- slow reads

---

## Fix

- rebuild indexes
- redesign indexing strategy
- partition large tables

---

# 5. Replication Lag Failures

## Definition

Read replicas fall behind the primary DB.

---

## Why it happens

- high write throughput
- slow replica nodes
- network delay

---

## Symptoms

- stale reads
- inconsistent data across services
- delayed dashboards/analytics

---

## Fix

- scale replicas
- optimize write load
- separate read/write workloads

---

# 6. Failover & Split-Brain Scenarios

## Definition

Multiple DB nodes believe they are primary.

---

## Why it happens

- network partition
- failed leader election
- misconfigured HA system

---

## Symptoms

- data inconsistency
- conflicting writes
- partial outage

---

## Fix

- enforce single leader election system
- use quorum-based systems
- strict fencing mechanisms

---

# 7. Connection Storm Cascades

## Definition

Retry storms cause exponential DB connection load.

---

## Failure Chain

    DB slow → retries increase → connection pool exhaustion → DB overload → full outage

---

## Symptoms

- sudden DB CPU spike
- cascading service failures
- exponential connection growth

---

## Fix

- implement retry backoff
- limit connection creation rate
- introduce circuit breakers

---

# 8. Real Production Incident Patterns

---

## Pattern 1: “Everything is slow”

Cause:
- DB latency increase propagating upstream

---

## Pattern 2: “Only one endpoint is broken”

Cause:
- slow query or missing index

---

## Pattern 3: “System-wide outage after traffic spike”

Cause:
- connection storm collapse

---

## Pattern 4: “Inconsistent data across services”

Cause:
- replication lag or split-brain

---

# 9. Debugging Flow

    Step 1: Check DB CPU / memory / IO
        ↓
    Step 2: Check slow query logs
        ↓
    Step 3: Check connection pool saturation
        ↓
    Step 4: Check replication lag
        ↓
    Step 5: Check application retry rate
        ↓
    Step 6: Identify amplification loop

---

# 10. Senior SRE Mental Model

Junior:
- “DB is slow”

Mid-level:
- “Check queries and indexes”

Senior:
- “Is this a query regression, connection storm, or replication lag issue?”

Principal:
- “Is the database acting as a bottleneck or an amplification point in a system-wide failure loop?”

---

# 11. Interview Answer Template

When diagnosing database-related failures, I first determine whether the issue is caused by query performance, connection saturation, replication lag, or locking contention. I check slow query logs and execution plans to identify inefficient queries or missing indexes. I then validate connection pool usage to detect saturation caused by high concurrency or retry storms. Next, I check replication health to ensure read consistency across replicas. If system-wide impact is observed, I look for feedback loops where slow database responses trigger retries and amplify load. I mitigate by reducing traffic load, optimizing queries, scaling database resources, and applying backpressure mechanisms, followed by long-term improvements in schema design, indexing strategy, and connection management.

---

# 12. Key Takeaways

- Database is the strongest coupling point in system design
- Query regressions are a major hidden failure source
- Connection storms cause exponential outages
- Replication lag breaks consistency silently
- Most “system outages” originate from DB bottlenecks

---

# End of Module 10
