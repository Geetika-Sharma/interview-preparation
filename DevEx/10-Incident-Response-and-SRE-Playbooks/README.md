# Incident Response and SRE Playbooks

## Introduction

Senior SRE and Platform Engineers are not measured only by how well they build systems.

They are measured by how well they respond when systems fail.

At scale, failures are expected:

- Hardware failures
- Network failures
- Bad deployments
- Dependency failures
- Capacity problems
- Human mistakes

The goal is not:

"Prevent every failure"

The goal is:

```
Detect Faster

Understand Faster

Recover Faster

Prevent Repeat Failures
```

---

# Incident Management

## Definition

Incident management is the structured process of responding to production issues.

A good incident process reduces:

- Downtime
- Confusion
- Communication gaps

---

# Incident Lifecycle

```
Detection

 |

Triage

 |

Mitigation

 |

Recovery

 |

Root Cause Analysis

 |

Prevention
```

---

# Incident Severity Levels

## Severity 1 (Critical)

Definition:

Major production impact.

Examples:

- Entire service unavailable
- Large customer impact
- Data loss risk

Response:

- Immediate escalation
- Incident commander assigned
- Frequent updates

---

## Severity 2 (High)

Definition:

Significant degradation.

Examples:

- Increased errors
- Partial outage
- Performance degradation

---

## Severity 3 (Medium)

Definition:

Limited impact.

Examples:

- Single feature failure
- Internal tooling issue

---

## Severity 4 (Low)

Definition:

Minor issue.

Examples:

- Documentation problem
- Non-critical bugs

---

# Incident Roles

## Incident Commander (IC)

Definition:

Person responsible for coordinating response.

Responsibilities:

- Assign owners
- Manage priorities
- Make decisions
- Coordinate communication

---

## Technical Lead

Responsible for:

- Debugging
- Root cause identification
- Technical mitigation

---

## Communication Lead

Responsible for:

- Stakeholder updates
- Status communication

---

# Incident Command Structure

```
Incident Commander

        |

 +------+------+

 |             |

Technical    Communication

Lead         Lead

 |

Engineering Teams
```

---

# Production Debugging Methodology

Senior engineers do not randomly execute commands.

They follow evidence.

---

# Debugging Framework

```
Understand Impact

        |

Gather Evidence

        |

Generate Hypothesis

        |

Test Hypothesis

        |

Mitigate

        |

Prevent
```

---

# Step 1: Understand Impact

Questions:

- What is broken?
- Who is affected?
- When did it start?
- Is impact increasing?

Example:

Bad:

"The API is slow."

Good:

"API latency increased from 200ms to 4 seconds after deployment at 10:30 UTC."

---

# Step 2: Gather Evidence

Sources:

- Metrics
- Logs
- Traces
- Events
- Recent changes

---

# Observability Signals

## Metrics

Definition:

Numeric measurements over time.

Examples:

- CPU
- Memory
- Latency
- Error rate

---

## Logs

Definition:

Detailed event records.

Examples:

- Application errors
- Authentication failures

---

## Traces

Definition:

Request journey across services.

Useful for:

- Microservices debugging
- Dependency failures

---

# The Four Golden Signals

Popular SRE monitoring model:

## Latency

How long requests take.

---

## Traffic

How much load exists.

---

## Errors

How many requests fail.

---

## Saturation

How close resources are to limits.

---

# The RED Method

Used for services.

## Rate

Requests per second.

---

## Errors

Failed requests.

---

## Duration

Request latency.

---

# The USE Method

Used for infrastructure.

## Utilization

Resource usage.

Example:

CPU 90%

---

## Saturation

Waiting or queueing.

Example:

CPU ready time.

---

## Errors

Hardware or system failures.

---

# Incident Example

## Problem

API latency increased.

---

## Investigation

Check:

```
Traffic

 |

Latency

 |

Errors

 |

Dependencies
```

---

Metrics:

```
Request latency increased

Database latency increased

CPU normal
```

Hypothesis:

Database bottleneck.

---

# Kubernetes Incident Playbook

# Pod CrashLoopBackOff

## Definition

Container starts, crashes, restarts repeatedly.

---

## Symptoms

```
kubectl get pods

STATUS:

CrashLoopBackOff
```

---

## Investigation

Check:

```
kubectl describe pod <pod>

kubectl logs <pod>

kubectl get events
```

---

## Possible Causes

### Application Error

Symptoms:

Application exits immediately.

Check:

Container logs.

---

### Missing Configuration

Symptoms:

Startup failure.

Check:

ConfigMaps and Secrets.

---

### Dependency Failure

Symptoms:

Application cannot connect.

Check:

Database, cache, APIs.

---

### Memory Limit Too Low

Symptoms:

OOMKilled.

Check:

Container memory usage.

---

# Immediate Mitigation

Options:

- Rollback deployment
- Increase resources
- Restore configuration
- Disable failing feature

---

# Long-Term Fix

Implement:

- Better health checks
- Resource tuning
- Automated testing
- Improved alerts

---

# Kubernetes Node Not Ready

## Definition

Node cannot run workloads normally.

---

## Symptoms

```
kubectl get nodes

STATUS:

NotReady
```

---

## Investigation

Check:

```
kubectl describe node <node>
```

Look for:

- Disk pressure
- Memory pressure
- Network issues

---

# AWS Incident Response

## Common AWS Failures

Examples:

- Availability Zone failure
- Network failure
- IAM permission issue
- Load balancer failure
- Storage failure

---

# AWS Debugging Approach

```
Application

 |

Load Balancer

 |

Compute

 |

Network

 |

Storage

 |

AWS Service Health
```

---

# Kafka Incident Playbook

# Consumer Lag

## Definition

Consumer lag measures how far consumers are behind producers.

---

## Symptoms

Messages accumulate.

---

## Investigation

Check:

- Consumer health
- Broker health
- Partition distribution

---

## Possible Causes

- Slow consumers
- Insufficient consumers
- Processing failures
- Rebalance events

---

# Database Saturation

## Symptoms

- High latency
- Connection failures
- Slow queries

---

## Investigation

Check:

- CPU
- Memory
- Connections
- Query latency

---

# Retry Storm

## Definition

A retry storm happens when many services retry failed requests simultaneously.

---

# Example

```
Service A

 |

Service B Failure

 |

Retry

 |

More Load

 |

Service B More Failure
```

---

# Prevention

Use:

- Exponential backoff
- Jitter
- Circuit breakers
- Retry limits

---

# Cascading Failure

## Definition

Failure in one component causes failures across dependent systems.

---

# Example

```
Database Slow

      |

API Slow

      |

Requests Timeout

      |

Retries Increase

      |

System Collapse
```

---

# Preventing Cascading Failures

Use:

- Timeouts
- Circuit breakers
- Rate limiting
- Bulkheads
- Load shedding

---

# Postmortem

## Definition

A postmortem is a document analyzing an incident.

The goal:

Learn and improve.

Not:

Assign blame.

---

# Good Postmortem Structure

## Summary

What happened?

---

## Impact

Who was affected?

---

## Timeline

What happened and when?

---

## Root Cause

Why did it happen?

---

## Resolution

How was it fixed?

---

## Prevention

How will recurrence be avoided?

---

# Root Cause Analysis

## Five Whys

Example:

Why did service fail?

Because database connections failed.

Why?

Connection pool exhausted.

Why?

Traffic increased.

Why?

No autoscaling.

Why?

Capacity planning missing.

---

# Senior SRE Interview Scenario

## Question

"Production latency increased by 10x. What do you do?"

---

## Senior Answer

"I would first understand impact and scope. I would check golden signals: latency, traffic, errors, and saturation. I would correlate with recent changes, deployments, and dependency health. I would prioritize mitigation first, then perform root cause analysis."

---

# Principal Engineer Thinking

A Principal Engineer considers:

- Immediate recovery
- System design flaws
- Organizational improvements
- Automation opportunities

Example:

Not only:

"Restart service"

But:

"Why did the system require manual restart?"

---

# Common Mistakes

## Mistake 1

Starting with fixes before understanding impact.

Problem:

Can worsen incidents.

---

## Mistake 2

Only checking application logs.

Problem:

Failures often occur in dependencies.

---

## Mistake 3

Ignoring recent changes.

Problem:

Most incidents correlate with changes.

---

# Incident Response Best Practices

Use:

- Clear ownership
- Runbooks
- Automated alerts
- Blameless reviews
- Production testing
- Capacity planning

---

# Key Takeaways

Senior incident response requires:

- Structured debugging
- Strong observability
- Clear communication
- Fast mitigation
- Learning from failures

The goal:

Build systems and teams that recover quickly from inevitable failures.

---
