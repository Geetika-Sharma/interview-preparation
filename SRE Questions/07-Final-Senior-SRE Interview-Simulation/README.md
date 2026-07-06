# 07 - Final Senior SRE Interview Simulation (Staff+ Level)

A full end-to-end simulation of real senior SRE interviews combining:
- Kubernetes
- AWS infrastructure
- Networking
- Cascading failures
- On-call reasoning
- System design under pressure

---

# Table of Contents

- Introduction
- How Senior SRE Interviews Are Structured
- Interview Round 1: Deep Kubernetes Debugging
- Interview Round 2: AWS + Infrastructure Failure
- Interview Round 3: Cross-Service Cascading Failure
- Interview Round 4: Networking + Latency Breakdown
- Interview Round 5: System Design + Reliability Tradeoffs
- Interview Round 6: Live On-Call Scenario (Full Simulation)
- Evaluation Criteria (What Interviewers Look For)
- Senior SRE Answer Framework
- Final Mental Model

---

# Introduction

At Staff / Senior SRE level, interviews are NOT:

- command memorization
- YAML writing
- basic Kubernetes questions

They ARE:

> “Can you reason about large distributed systems under ambiguity and failure?”

---

# How Senior SRE Interviews Are Structured

You will be tested on:

- Debugging unknown production issues
- Multi-layer failure reasoning
- Tradeoff decisions under constraints
- AWS + Kubernetes interaction understanding
- Incident leadership

---

# Interview Round 1: Deep Kubernetes Debugging

## Prompt

A deployment is stuck in `Pending` across a 500-node Kubernetes cluster.

---

## Expected Thinking

You should immediately classify:

### Step 1: Check Pod Events
- scheduling failure reason

### Step 2: Identify constraint type
- CPU / Memory
- Node selector
- Taints / tolerations
- PVC
- Topology spread

---

## Strong Answer Pattern

- Identify scheduling predicate failure
- Confirm cluster-wide vs partial issue
- Check node availability and constraints
- Validate autoscaler behavior

---

## Follow-up Trap

Interviewer may say:

> “CPU usage is only 50% cluster-wide”

Correct insight:

> This is NOT a resource issue — it is a constraint fragmentation problem.

---

# Interview Round 2: AWS + Infrastructure Failure

## Prompt

Pods cannot be scheduled in one AZ, but others are fine.

---

## Expected Reasoning

- EC2 capacity mismatch
- AZ-level instance exhaustion
- EBS volume AZ binding issue
- Node group imbalance

---

## Strong Answer

- Isolate AZ-level failure
- Validate EC2 capacity constraints
- Check node group distribution
- Confirm storage locality constraints

---

# Interview Round 3: Cross-Service Cascading Failure

## Prompt

One service latency increase caused system-wide slowdown.

---

## Expected Reasoning

- Identify retry amplification
- Check dependency chain
- Look for circuit breaker absence
- Validate downstream bottlenecks (DB, Kafka)

---

## Strong Answer

- Detect feedback loop
- Identify retry storm
- Break dependency chain
- Stop amplification first, debug later

---

# Interview Round 4: Networking + Latency Breakdown

## Prompt

Cross-service calls fail intermittently with high latency.

---

## Expected Reasoning

Check layers:

- DNS (CoreDNS)
- CNI networking
- kube-proxy routing
- AWS VPC / NAT
- Security groups

---

## Strong Answer

- Identify layer of failure
- Confirm whether issue is packet loss or routing
- Isolate AZ-specific network degradation
- Validate NAT / DNS bottlenecks

---

# Interview Round 5: System Design + Reliability Tradeoffs

## Prompt

Design a highly reliable system handling:

- 100B+ requests/day
- multi-region deployment
- failure isolation
- cost optimization

---

## Expected Focus Areas

- Horizontal scaling
- Multi-AZ architecture
- Stateless services
- Queue-based decoupling
- Backpressure handling
- Observability

---

## Strong Answer Pattern

- Design for failure, not perfection
- Introduce isolation boundaries
- Reduce synchronous dependencies
- Add retry + circuit breaker strategy
- Use autoscaling carefully

---

# Interview Round 6: Live On-Call Simulation

## Scenario

PagerDuty alert:

> “Severe latency spike across multiple services + deployment failures”

---

## Step 1: Immediate Classification

- Is it:
  - Kubernetes issue?
  - AWS degradation?
  - Network failure?
  - Cascading retry storm?

---

## Step 2: First Actions

- Check dashboards
- Check pod health
- Check error rate vs latency correlation

---

## Step 3: Likely Diagnosis Paths

### Path A: Retry Storm
- traffic amplification

### Path B: DNS failure
- CoreDNS overload

### Path C: AWS degradation
- EC2 / VPC / NAT issues

### Path D: Scheduling failure
- cluster capacity issue

---

## Step 4: Mitigation First

- Stop traffic amplification
- Scale critical services
- Isolate failing dependency

---

## Step 5: Deep Debug

- trace dependency chain
- identify root layer
- confirm systemic cause

---

# Evaluation Criteria (What Interviewers Look For)

## 1. Signal Identification
Can you find the *right layer quickly*?

## 2. System Thinking
Do you understand dependencies?

## 3. Failure Containment
Do you stop the bleeding first?

## 4. Tradeoff Reasoning
Can you balance speed vs correctness?

## 5. Seniority Signal
Do you think in systems, not components?

---

# Senior SRE Answer Framework

Use this structure every time:

### 1. Classify problem domain
- scheduling / networking / storage / AWS / cascading

### 2. Identify blast radius
- single pod / service / cluster / region

### 3. Check control plane signals
- events, metrics, logs

### 4. Validate infrastructure layer
- nodes, networking, AWS dependencies

### 5. Mitigate first
- stop amplification, restore service

### 6. Deep root cause analysis
- after stabilization

### 7. Prevent recurrence
- architecture + policy changes

---

# Final Mental Model

At Staff SRE level:

You are NOT debugging pods.

You are debugging:

- distributed systems behavior
- failure propagation
- system-wide coupling
- infrastructure dependencies

---

# Final Takeaways

- Most failures are multi-layer, not single point
- Fast classification > deep early debugging
- Mitigation always comes before RCA
- AWS + Kubernetes failures are deeply coupled
- Senior engineers think in systems, not components

---

# END OF FULL SERIES

You now have a complete simulation of:

- Kubernetes scheduling failures
- Networking breakdowns
- Cascading system failures
- AWS infrastructure failures
- On-call incident response
- Senior SRE interview reasoning

---
