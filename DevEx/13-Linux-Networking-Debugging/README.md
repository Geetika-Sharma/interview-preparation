# Linux Networking and Debugging

## Introduction

Senior Platform Engineers spend significant time debugging failures below the application layer.

Many production problems are caused by:

- Operating system limits
- CPU saturation
- Memory exhaustion
- Disk pressure
- Network failures
- DNS issues
- Kernel behavior

A strong SRE understands:

```
Application

 |

Runtime

 |

Operating System

 |

Kernel

 |

Hardware
```

---

# Linux Architecture Basics

## Linux Kernel

Definition:

The kernel is the core component that manages:

- CPU
- Memory
- Processes
- Devices
- Networking

---

## User Space

Definition:

Applications running outside the kernel.

Examples:

- Java application
- Nginx
- Kubernetes containers

---

## Kernel Space

Definition:

Privileged area where the operating system manages hardware resources.

---

# Linux Process Model

## Process

Definition:

A running instance of a program.

Example:

```
Application

 |

Process ID (PID)
```

---

## Thread

Definition:

A smaller execution unit inside a process.

Example:

A web server process may have:

```
1 Process

100 Threads
```

---

# Process Troubleshooting

## Check Running Processes

Command:

```
ps aux
```

Purpose:

Shows running processes.

Useful when:

- CPU is high
- Memory is high
- Application is stuck

---

## Find Process Details

Command:

```
top
```

Purpose:

Real-time process monitoring.

Look for:

- CPU usage
- Memory usage
- Process state

---

## Advanced Process Monitoring

Command:

```
htop
```

Benefits:

- Better visualization
- Easier process filtering

---

# CPU Troubleshooting

## CPU Utilization

Definition:

Percentage of CPU being used.

---

High CPU symptoms:

- Slow application response
- Increased latency
- Request failures

---

# CPU Investigation Flow

```
High CPU

 |

Which Process?

 |

Application or System?

 |

CPU Profile

 |

Root Cause
```

---

## Commands

### top

Check:

```
CPU usage

Load average

Processes
```

---

### pidstat

Command:

```
pidstat -u
```

Purpose:

Shows CPU usage per process.

---

### uptime

Shows:

- Load average
- System pressure

Example:

```
load average:

10.5 10.2 9.8
```

---

# CPU Load Average

Definition:

Number of processes waiting for CPU or resources.

High load does not always mean high CPU.

Example:

CPU:

```
40%
```

Load:

```
100
```

Possible cause:

Disk or IO wait.

---

# Memory Troubleshooting

## Memory

Definition:

RAM used by running processes.

---

Memory problems cause:

- Application crashes
- OOM kills
- Performance issues

---

# Memory Investigation

Command:

```
free -m
```

Shows:

- Used memory
- Available memory
- Cache

---

## Process Memory

Command:

```
ps aux --sort=-%mem
```

Finds:

Largest memory consumers.

---

# OOM Killer

## Definition

Linux component that terminates processes when memory is exhausted.

---

Example:

```
Memory exhausted

 |

Kernel selects process

 |

Process killed
```

---

# OOM Symptoms

Application:

```
Unexpected restart
```

Logs:

```
Killed process
```

Kubernetes:

```
OOMKilled
```

---

# Prevention

Use:

- Memory limits
- Capacity planning
- Leak detection
- Monitoring

---

# Disk Troubleshooting

## Disk Pressure

Definition:

System does not have enough storage capacity.

---

Symptoms:

- Application failures
- Log write failures
- Kubernetes DiskPressure

---

# Disk Commands

## df

Command:

```
df -h
```

Purpose:

Shows filesystem usage.

---

## du

Command:

```
du -sh *
```

Purpose:

Finds large directories.

---

## iostat

Command:

```
iostat
```

Purpose:

Shows disk performance.

Metrics:

- Utilization
- Latency
- Throughput

---

# Network Fundamentals

## IP Address

Definition:

Identifier used to communicate on networks.

---

## Port

Definition:

Logical endpoint for network communication.

Example:

```
Application

Port 8080
```

---

## TCP

Definition:

Reliable communication protocol.

Provides:

- Connection management
- Ordering
- Retransmission

---

## UDP

Definition:

Faster connectionless protocol.

Used for:

- DNS
- Streaming
- Real-time communication

---

# TCP Connection Lifecycle

```
Client

 |

SYN

 |

Server

 |

SYN ACK

 |

Client

 |

ACK

 |

Connection Established
```

---

# Common Network Problems

## Connection Timeout

Meaning:

Client cannot establish connection.

Possible causes:

- Firewall
- Network route
- Service unavailable

---

## Connection Refused

Meaning:

Server reachable but no application listening.

Example:

Wrong port.

---

## Packet Loss

Meaning:

Network packets disappear.

Symptoms:

- Slow requests
- Retries
- Timeouts

---

# Network Troubleshooting Commands

---

# ping

Purpose:

Tests basic connectivity.

Example:

```
ping server
```

Useful for:

- Reachability testing

Limitations:

Does not prove application availability.

---

# traceroute

Purpose:

Shows network path.

Useful for:

- Routing problems
- Network hops

---

# curl

Purpose:

Tests HTTP connectivity.

Example:

```
curl -v https://service
```

Shows:

- Connection
- TLS
- HTTP response

---

# ss

Command:

```
ss -tulpn
```

Purpose:

Shows listening ports.

Useful for:

- Service availability
- Port debugging

---

# tcpdump

Definition:

Packet capture tool.

Example:

```
tcpdump port 443
```

Used when:

- Network behavior is unclear
- Application sees timeouts

---

# DNS Troubleshooting

## DNS

Definition:

System that converts names into IP addresses.

Example:

```
api.example.com

        |

        v

10.0.1.20
```

---

# DNS Resolution Flow

```
Application

 |

Resolver

 |

DNS Server

 |

IP Address
```

---

# DNS Failures

Symptoms:

- Application cannot connect
- Random failures
- Service discovery problems

---

# DNS Commands

## nslookup

Purpose:

Query DNS records.

---

## dig

Example:

```
dig service.example.com
```

Shows:

- DNS records
- Response time
- Nameservers

---

# Kubernetes Networking Debugging

## Service Discovery Flow

```
Application

 |

Service Name

 |

CoreDNS

 |

Service IP

 |

Pod IP
```

---

# Kubernetes Network Issues

Common causes:

- DNS failure
- CNI failure
- Network policy
- Service misconfiguration

---

# Debugging Steps

Check:

```
kubectl get svc

kubectl get endpoints

kubectl exec pod -- nslookup service
```

---

# Production Scenario

## Incident

Users report:

"API requests randomly timeout."

---

# Investigation

Check:

## Application

Metrics:

- Error rate
- Latency

---

## Network

Check:

- Packet loss
- Connection errors

---

## Infrastructure

Check:

- CPU
- Memory
- Nodes

---

Finding:

Database connections timing out because connection pool exhausted.

---

# Advanced Linux Debugging

## strace

Definition:

Tracks system calls made by a process.

Example:

```
strace -p PID
```

Useful for:

- Hanging processes
- File access problems

---

## lsof

Definition:

Lists open files.

Example:

```
lsof -i :8080
```

Useful for:

- Port ownership
- File leaks

---

## vmstat

Shows:

- CPU
- Memory
- IO statistics

---

## sar

System activity reporting.

Useful for:

- Historical analysis

---

# Senior Debugging Methodology

Junior approach:

```
Restart service
```

---

Senior approach:

```
Understand Impact

 |

Collect Evidence

 |

Create Hypothesis

 |

Validate

 |

Mitigate

 |

Prevent
```

---

# Interview Questions

## Easy

### How do you troubleshoot high CPU?

Answer:

"I identify the consuming process, determine whether CPU usage is expected, analyze application behavior, and check recent changes."

---

## Medium

### Application is slow but CPU is normal. What do you check?

Answer:

"I would investigate memory pressure, network latency, database performance, external dependencies, and IO wait."

---

## Hard

### DNS works from one server but not another. How do you debug?

Answer:

"I would compare resolver configuration, DNS servers, network policies, caching behavior, and query responses."

---

## Staff Level

### How do you build Linux debugging capability across teams?

Answer:

"I would create operational standards: observability, runbooks, automation, training, and self-service diagnostics."

---

# Common Mistakes

## Mistake 1

Only checking application logs.

Problem:

Root cause may be infrastructure.

---

## Mistake 2

Restarting before collecting evidence.

Problem:

Destroys investigation data.

---

## Mistake 3

Assuming CPU is the only performance problem.

Problem:

Latency can come from memory, IO, network, or dependencies.

---

# Key Takeaways

Senior Linux troubleshooting requires understanding:

- Processes
- CPU
- Memory
- Disk
- Networking
- DNS
- Kernel behavior
- System observability

The goal:

Move from guessing to evidence-based debugging.
