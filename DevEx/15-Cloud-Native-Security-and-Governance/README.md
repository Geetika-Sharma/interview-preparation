# Cloud-Native Security and Governance

## Introduction

Senior Platform Engineers are responsible for building platforms that enable developers while maintaining security, compliance, and operational control.

A modern cloud platform must balance:

```
Developer Velocity

        +

Security

        +

Reliability

        +

Governance
```

The goal is not to block engineers.

The goal is:

```
Secure Defaults

+

Self-Service

+

Automation
```

---

# Security Principles

## Least Privilege

Definition:

Users and systems should receive only the permissions required to perform their tasks.

Example:

Bad:

```
Developer

 |

Administrator Access
```

Better:

```
Developer

 |

Limited Deployment Permissions
```

---

## Defense in Depth

Definition:

Using multiple security layers instead of depending on one control.

Example:

```
Identity Control

        |

Network Control

        |

Application Security

        |

Runtime Security
```

---

## Security by Default

Definition:

New systems should start with secure configurations.

Example:

Default:

```
Private Network

Encryption Enabled

Logging Enabled
```

---

# Cloud Security Model

Cloud security responsibilities are shared.

```
Cloud Provider

 |

Physical Infrastructure

 |

Customer

 |

Applications

Data

Configuration
```

---

# Identity and Access Management (IAM)

## Definition

IAM controls:

- Who can access resources
- What actions they can perform
- Which resources they can access

---

# IAM Components

## User

Definition:

An identity representing a person.

Example:

```
Engineer Account
```

---

## Role

Definition:

Temporary permissions assumed by users or services.

Preferred over long-lived credentials.

Example:

```
Application

 |

Assume Role

 |

Access AWS Resource
```

---

## Policy

Definition:

A document defining allowed or denied actions.

Example:

```
Allow:

Read S3 Bucket

Deny:

Delete Production Data
```

---

# AWS IAM Best Practices

Use:

- Roles instead of access keys
- Short-lived credentials
- MFA
- Least privilege policies
- Permission boundaries

Avoid:

- Shared accounts
- Long-lived credentials
- Administrator access

---

# Multi-Account AWS Governance

## Why Multiple Accounts?

Large organizations separate workloads.

Example:

```
Organization

 |

+----------------+

Security Account

 |

Logging Account

 |

Production Accounts

 |

Development Accounts
```

---

# Benefits

## Security Isolation

Production resources are separated.

---

## Cost Visibility

Teams understand ownership.

---

## Access Control

Different permissions per environment.

---

# AWS Organizations

## Definition

Service used to manage multiple AWS accounts centrally.

Used for:

- Account management
- Policies
- Governance

---

# Service Control Policies (SCP)

## Definition

Organization-level policies that define maximum permissions.

Example:

```
Deny

Creating Public S3 Buckets
```

Important:

SCPs do not grant permissions.

They limit permissions.

---

# AWS Networking Security

## VPC

Definition:

A private network environment inside AWS.

---

# Network Security Layers

```
Internet

 |

Load Balancer

 |

Public Subnet

 |

Private Subnet

 |

Application

 |

Database
```

---

# Security Groups

Definition:

Virtual firewall controlling traffic to resources.

Example:

Allow:

```
Application Server

to

Database Port 5432
```

---

# Network ACLs

Definition:

Subnet-level traffic filtering.

Used for:

- Additional network controls
- Compliance requirements

---

# Kubernetes Security

Kubernetes security has multiple layers:

```
Cluster

 |

API Server

 |

Identity

 |

RBAC

 |

Workloads

 |

Containers
```

---

# Kubernetes RBAC

## Definition

Role-Based Access Control controls who can perform Kubernetes actions.

Example:

Developer:

```
Can deploy applications

Cannot delete namespaces
```

---

# RBAC Components

## Role

Defines permissions inside a namespace.

---

## ClusterRole

Defines cluster-wide permissions.

---

## RoleBinding

Connects users to roles.

---

# Kubernetes Security Best Practices

Use:

- Namespace isolation
- RBAC
- Network policies
- Pod security controls
- Secret management
- Image scanning

---

# Kubernetes Secrets

## Definition

Objects storing sensitive information.

Examples:

- Passwords
- Tokens
- Certificates

---

# Secret Risks

Problem:

Kubernetes secrets are not automatically secure.

Risks:

- Incorrect permissions
- Accidental exposure
- Logging secrets

---

# Better Approach

Use:

- External secret managers
- Encryption at rest
- Access controls

Examples:

- AWS Secrets Manager
- HashiCorp Vault

---

# Container Security

## Image Security

Risks:

- Vulnerable packages
- Malware
- Outdated dependencies

---

# Secure Image Pipeline

```
Developer

 |

Build Image

 |

Security Scan

 |

Approval

 |

Registry

 |

Deployment
```

---

# Image Scanning

Checks:

- CVEs
- Package vulnerabilities
- Configuration issues

---

# Runtime Security

Protect:

- Containers
- Hosts
- Kubernetes workloads

Monitor:

- Unexpected processes
- Privilege escalation
- Network behavior

---

# CI/CD Security

CI/CD pipelines are production systems.

Security must protect:

- Source code
- Credentials
- Build systems
- Deployment processes

---

# Pipeline Security Risks

## Exposed Secrets

Example:

```
AWS_ACCESS_KEY=value
```

Problem:

Credential leakage.

---

## Dependency Vulnerabilities

Example:

Application uses vulnerable package.

---

## Unsafe Permissions

Example:

Pipeline has:

```
Administrator Access
```

---

# Secure CI/CD Practices

Use:

- Secret managers
- Short-lived credentials
- Branch protection
- Required reviews
- Security scanning
- Signed artifacts

---

# GitHub Actions Security

Important controls:

## Permissions

Restrict workflow permissions.

Example:

```
Read repository

Not

Write everything
```

---

## Environment Protection

Use approvals for:

- Production deployments

---

## Reusable Workflows

Benefits:

- Standard security controls
- Consistent pipelines

---

# Terraform Security

## Infrastructure as Code Risk

Terraform can create:

- Networks
- IAM policies
- Databases
- Production infrastructure

Mistakes can have large impact.

---

# Terraform Security Practices

## Code Review

All changes reviewed before applying.

---

## State Protection

Terraform state contains sensitive information.

Protect with:

- Encryption
- Access control
- Secure backend

---

## Policy as Code

Definition:

Automated rules that validate infrastructure.

Example:

Reject:

```
Public Database
```

Allow:

```
Private Database
```

---

# Infrastructure Drift

## Definition

When actual infrastructure differs from code.

Example:

Terraform:

```
Database Private
```

Actual:

```
Database Public
```

---

# Drift Prevention

Use:

- Regular plans
- Automation
- Restricted manual changes

---

# Compliance and Governance

## Definition

Rules and controls ensuring systems meet organizational requirements.

Examples:

- Security standards
- Audit requirements
- Data protection

---

# Platform Governance Model

A mature platform provides:

```
Guardrails

+

Self-Service

+

Visibility
```

---

# Example Platform Guardrails

Automatically enforce:

- Required tags
- Encryption
- Approved regions
- Security policies
- Resource ownership

---

# Golden Paths

## Definition

Recommended secure ways to build and deploy applications.

Example:

Developer creates service:

Gets automatically:

- Repository
- CI pipeline
- Security scanning
- Kubernetes deployment
- Monitoring

---

# Developer Experience vs Security

Bad security:

```
Developer

 |

Submit Ticket

 |

Wait Days
```

---

Better:

```
Developer

 |

Self-Service Platform

 |

Secure Defaults
```

---

# Incident Security Scenario

## Problem

A production credential is exposed.

---

# Immediate Response

Steps:

1. Revoke credential
2. Identify usage
3. Check logs
4. Rotate secrets
5. Investigate impact

---

# Long-Term Prevention

Implement:

- Secret scanning
- Better IAM
- Short-lived credentials
- Developer education

---

# Supply Chain Security

## Definition

Protecting software from source code to production deployment.

---

# Supply Chain Flow

```
Source Code

 |

Build

 |

Dependencies

 |

Container Image

 |

Deployment

 |

Runtime
```

---

# Security Controls

Use:

- Dependency scanning
- Artifact signing
- Image scanning
- Provenance tracking

---

# AI Security Considerations

AI-native platforms introduce new risks.

---

# AI Data Exposure

Risk:

Sensitive information sent to external AI tools.

Controls:

- Approved AI tools
- Data policies
- Enterprise controls

---

# AI Generated Code Risk

Risk:

AI produces insecure code.

Controls:

- Code review
- Security scanning
- Testing

---

# AI Automation Risk

Risk:

AI agent performs unsafe actions.

Controls:

- Permission limits
- Approval workflows
- Audit logs

---

# Production Scenario

## Incident

A developer accidentally deploys an insecure Kubernetes workload.

---

## Detection

Security tools identify:

- Privileged container
- Public exposure

---

## Response

Actions:

- Block deployment
- Notify team
- Fix configuration

---

## Prevention

Add:

- Admission policies
- Secure templates
- Automated checks

---

# Interview Questions

## Easy

### What is least privilege?

Answer:

"Giving users and systems only the permissions required to complete their tasks."

---

## Medium

### How do you secure Kubernetes?

Answer:

"I secure Kubernetes through RBAC, network policies, workload security, image scanning, secrets management, and strong access controls."

---

## Hard

### How do you balance developer velocity and security?

Answer:

"I prefer automated guardrails and self-service secure patterns rather than manual approval processes that slow teams down."

---

## Staff Level

### How would you build security into a developer platform?

Answer:

"I would make security part of the platform defaults: secure templates, automated scanning, policy enforcement, identity controls, and continuous compliance monitoring."

---

# Common Mistakes

## Mistake 1

Giving developers administrator access.

Problem:

Creates unnecessary risk.

---

## Mistake 2

Treating security as a separate team responsibility.

Problem:

Security must be embedded into platform design.

---

## Mistake 3

Manual governance processes.

Problem:

Do not scale with engineering growth.

---

# Production Best Practices

Use:

- Identity-first security
- Automated controls
- Infrastructure as Code
- Policy enforcement
- Continuous monitoring
- Secure developer workflows

---

# Key Takeaways

Cloud-native security is not about adding blockers.

A mature platform provides:

```
Fast Development

+

Secure Defaults

+

Automated Governance

+

Operational Visibility
```

Senior Platform Engineers build systems where security and developer productivity improve together.
