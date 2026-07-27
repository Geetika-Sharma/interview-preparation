# AWS Infrastructure Platform Engineering

## Introduction

Senior Platform Engineers are expected to understand AWS beyond individual services.

The focus is not:

"How do I create an EC2 instance?"

The focus is:

"How do I design, automate, secure, and operate cloud infrastructure at scale?"

A mature AWS platform provides:

- Reliable infrastructure
- Standardized environments
- Security controls
- Cost governance
- Developer self-service
- Operational visibility

---

# AWS Platform Architecture Overview

A typical enterprise AWS environment contains:

```
Users

 |

Identity Provider

 |

AWS Accounts

 |

VPC Network

 |

Compute

 |

Applications

 |

Monitoring
```

---

# AWS Account Strategy

## Definition

An AWS account is an isolated security and billing boundary.

Organizations usually separate accounts by:

- Environment
- Team
- Application
- Security boundary

---

# Multi Account Model

Example:

```
Organization

 |

 +-- Security Account

 +-- Logging Account

 +-- Network Account

 +-- Development Accounts

 +-- Production Accounts
```

---

# Why Multiple AWS Accounts?

Benefits:

| Benefit | Reason |
|---|---|
| Isolation | Limits blast radius |
| Security | Separate permissions |
| Billing | Better cost tracking |
| Compliance | Easier auditing |

---

# AWS Organizations

## Definition

AWS Organizations manages multiple AWS accounts centrally.

Capabilities:

- Account creation
- Policy enforcement
- Billing management
- Governance

---

# Service Control Policies (SCP)

## Definition

SCPs define maximum permissions allowed inside AWS accounts.

Important:

SCPs do not grant permissions.

They restrict permissions.

---

Example:

Account policy:

```
Allow EC2 Creation
```

SCP:

```
Deny Public S3 Buckets
```

Result:

Public buckets cannot be created.

---

# AWS Identity and Access Management (IAM)

## Definition

IAM controls who can access AWS resources and what actions they can perform.

IAM manages:

- Users
- Roles
- Policies
- Permissions

---

# IAM Best Practice

Avoid:

```
Application

 |

Hardcoded Access Keys
```

Prefer:

```
Application

 |

IAM Role

 |

AWS Service
```

---

# IAM Role

## Definition

A role provides temporary permissions that can be assumed by users or services.

Common uses:

- EC2 access
- Kubernetes workloads
- CI/CD pipelines
- Automation

---

# Least Privilege

## Definition

Give only required permissions.

Bad:

```
AdministratorAccess
```

for every engineer.

Better:

```
Developer:

Deploy Application


Platform:

Manage Infrastructure
```

---

# IAM Policy Structure

A policy contains:

- Effect
- Action
- Resource

Example:

```
Allow

s3:GetObject

specific bucket
```

---

# AWS Networking

## Introduction

Networking is one of the most important AWS interview areas.

Senior engineers should understand:

- VPC
- Subnets
- Routing
- Security Groups
- Load Balancers
- DNS
- NAT
- Connectivity

---

# VPC

## Definition

A Virtual Private Cloud is an isolated AWS network.

A VPC contains:

- IP ranges
- Subnets
- Routing
- Security controls

---

# Subnet

## Definition

A subnet is a smaller network segment inside a VPC.

Common types:

## Public Subnet

Has direct internet access.

Used for:

- Load balancers
- Public services

---

## Private Subnet

No direct internet access.

Used for:

- Applications
- Databases
- Internal services

---

# Availability Zones

## Definition

Availability Zones are separate AWS data center locations inside a region.

Applications use multiple AZs for reliability.

Example:

```
Region

 |

 +-- AZ A

 |

 +-- AZ B

 |

 +-- AZ C
```

---

# Route Table

## Definition

A route table controls where network traffic goes.

Example:

```
Application Subnet

 |

Route Table

 |

NAT Gateway

 |

Internet
```

---

# Security Group

## Definition

A Security Group is a virtual firewall attached to resources.

Controls:

- Allowed inbound traffic
- Allowed outbound traffic

---

Example:

Application server:

Allow:

```
Port 443

from Load Balancer
```

Deny:

```
Direct internet access
```

---

# Network ACL

## Definition

Network ACLs are subnet-level firewall rules.

Difference:

| Security Group | Network ACL |
|-|-|
| Resource level | Subnet level |
| Stateful | Stateless |

---

# AWS Load Balancers

## Definition

Load balancers distribute traffic across multiple targets.

---

# Application Load Balancer (ALB)

Used for:

- HTTP
- HTTPS
- Path routing
- Host routing

Example:

```
/api

 |

API Service


/web

 |

Frontend Service
```

---

# Network Load Balancer (NLB)

Used for:

- High performance
- TCP/UDP traffic
- Low latency workloads

---

# ALB vs NLB

| ALB | NLB |
|-|-|
| Layer 7 | Layer 4 |
| HTTP aware | TCP aware |
| Routing rules | High throughput |

---

# AWS DNS

## Route 53

Definition:

AWS managed DNS service.

Used for:

- Domain management
- Routing traffic
- Health checks

---

# DNS Failure Scenario

Symptoms:

- Applications cannot resolve domains
- Increased connection failures

Investigation:

Check:

- DNS records
- Health checks
- Resolver configuration

---

# NAT Gateway

## Definition

NAT Gateway allows private resources to access the internet without being publicly accessible.

Example:

```
Private Instance

 |

NAT Gateway

 |

Internet
```

---

# AWS Compute Services

## EC2

Definition:

Virtual machines running in AWS.

Used for:

- Applications
- Infrastructure workloads

---

## Auto Scaling Group

Definition:

Automatically adjusts EC2 instance count.

Based on:

- CPU
- Memory
- Custom metrics

---

# Container Infrastructure

Modern platforms commonly use:

- EKS
- ECS
- Fargate

---

# AWS Elastic Kubernetes Service (EKS)

## Definition

EKS is AWS managed Kubernetes.

AWS manages:

- Kubernetes control plane

Customer manages:

- Worker nodes
- Applications
- Add-ons

---

# EKS Architecture

```
EKS Cluster

 |

 +-- Control Plane

 |

 +-- Worker Nodes

 |

 +-- Pods
```

---

# EKS Platform Responsibilities

Platform teams manage:

- Cluster lifecycle
- Node groups
- IAM integration
- Networking
- Add-ons
- Monitoring
- Security

---

# EKS Networking

Important components:

- VPC CNI
- Security Groups
- Load Balancers
- Network Policies

---

# IAM and EKS

A major production pattern:

```
Kubernetes Service Account

 |

IAM Role

 |

AWS Resource Access
```

This avoids static credentials.

---

# Infrastructure as Code

## Definition

Infrastructure as Code manages infrastructure using version-controlled files.

Examples:

- Terraform
- CloudFormation
- CDK

---

# Terraform Workflow

```
Code

 |

terraform plan

 |

Review

 |

terraform apply

 |

Infrastructure
```

---

# Terraform State

## Definition

Terraform state stores knowledge of deployed resources.

Important because Terraform needs to know:

Desired:

```
Infrastructure Code
```

Current:

```
AWS Resources
```

---

# Terraform Production Practices

Use:

- Remote state
- State locking
- Code review
- Modules
- CI validation

---

# AWS Cost Optimization

## Introduction

Platform engineers are responsible for efficient infrastructure usage.

---

# Common Cost Problems

## Oversized Resources

Example:

Application needs:

```
2 CPU
```

but runs:

```
16 CPU instance
```

---

## Idle Resources

Examples:

- Unused load balancers
- Old snapshots
- Unused IP addresses

---

## Poor Scaling

Example:

Running maximum capacity all day.

---

# Cost Optimization Techniques

## Right Sizing

Match resources to workload requirements.

---

## Auto Scaling

Scale with demand.

---

## Reserved Capacity

Use commitments for stable workloads.

---

## Resource Tagging

Track ownership and cost.

---

# AWS Production Incident Scenarios

# Incident: Application Cannot Reach Database

## Symptoms

Application errors:

```
Connection timeout
```

---

## Investigation

Check:

1. Security Groups
2. Network routes
3. DNS
4. Database availability
5. Application configuration

---

# Incident: High AWS Latency

Possible causes:

- Network congestion
- Load balancer issues
- Application saturation
- Dependency failures

---

Investigation:

Check:

- CloudWatch metrics
- Load balancer metrics
- Application traces

---

# Incident: EC2 Instance Unreachable

Check:

1. Instance status checks
2. Security groups
3. Route tables
4. Network interface
5. OS health

---

# Senior AWS Interview Questions

## Question

How would you design AWS infrastructure for hundreds of applications?

---

## Senior Answer

"I would create a standardized cloud platform using multi-account architecture, infrastructure as code, centralized security controls, reusable networking patterns, observability, and self-service workflows."

---

## Question

How do you secure AWS access?

---

Answer:

"I use identity-based access, IAM roles, least privilege policies, centralized authentication, audit logging, and avoid long-lived credentials."

---

# AWS Platform Engineering Summary

A Senior Platform Engineer should understand:

## Cloud Foundation

- Accounts
- Organizations
- IAM
- Governance

## Networking

- VPC
- Subnets
- Routing
- Load balancers
- DNS

## Compute

- EC2
- Containers
- EKS

## Automation

- Terraform
- CI/CD
- Self-service infrastructure

## Operations

- Monitoring
- Incident response
- Cost optimization

---

# Final Interview Statement

"Cloud platform engineering is about creating a secure, scalable, and repeatable foundation that allows engineering teams to deliver applications quickly without needing to understand every infrastructure detail."

---
