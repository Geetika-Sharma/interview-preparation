# GitOps and ArgoCD Platform Engineering

## Introduction

Modern Kubernetes platforms require reliable deployment automation.

Traditional deployment:

```
Engineer

 |

kubectl apply

 |

Production Cluster
```

Problems:

- Manual changes
- Poor auditability
- Configuration drift
- Difficult rollback

GitOps changes the model:

```
Git Repository

 |

GitOps Controller

 |

Kubernetes Cluster
```

Git becomes the source of truth.

---

# What is GitOps?

## Definition

GitOps is an operational model where:

- Desired infrastructure state is stored in Git.
- Automated controllers reconcile Git state with runtime state.

The cluster continuously moves toward the desired state.

---

# GitOps Principles

| Principle | Meaning |
|---|---|
| Declarative | Describe desired state |
| Version Controlled | Every change has history |
| Automated Reconciliation | System fixes drift |
| Pull Based Delivery | Cluster pulls changes |

---

# GitOps vs Traditional CI/CD

## Traditional CI/CD

```
Developer

 |

Pipeline

 |

kubectl deploy

 |

Cluster
```

Pipeline has direct cluster access.

---

## GitOps

```
Developer

 |

Git Commit

 |

Repository

 |

ArgoCD

 |

Cluster
```

Cluster credentials are not exposed to pipelines.

---

# GitOps Repository Structure

Example:

```
gitops-repository

 |

 +-- applications

 |

 +-- environments

       |

       +-- dev

       |

       +-- staging

       |

       +-- production
```

---

# ArgoCD

## Definition

ArgoCD is a Kubernetes GitOps continuous delivery controller.

It:

- Watches Git repositories
- Compares desired and actual state
- Synchronizes changes
- Reports health

---

# ArgoCD Architecture

```
Git Repository

      |

      v

ArgoCD Controller

      |

      v

Kubernetes API Server

      |

      v

Resources
```

---

# ArgoCD Components

## API Server

Provides:

- UI
- CLI access
- Authentication

---

## Application Controller

Responsible for:

- Comparing Git and cluster state
- Sync decisions
- Health checks

---

## Repository Server

Handles:

- Git access
- Manifest generation

---

# ArgoCD Application

## Definition

An Application object connects:

- Git repository
- Kubernetes cluster
- Deployment path

Example:

```
Application

Source:

Git Repository


Destination:

Production Cluster
```

---

# Sync States

## Synced

Git and cluster match.

---

## OutOfSync

Difference exists.

Example:

Git:

```
replicas: 5
```

Cluster:

```
replicas: 3
```

---

## Unknown

ArgoCD cannot determine state.

Possible causes:

- API failure
- Permission issue
- Network problem

---

# Application Health

## Healthy

Application running normally.

---

## Progressing

Deployment is changing.

---

## Degraded

Application is unhealthy.

Examples:

- Pods failing
- Readiness failures
- Deployment stuck

---

# ArgoCD Sync Policies

## Manual Sync

Engineer approves deployment.

Used for:

- Production changes
- Sensitive workloads

---

## Automated Sync

ArgoCD automatically applies Git changes.

Used for:

- Lower environments
- Trusted workflows

---

# Automated Sync Options

## Self Heal

Definition:

ArgoCD automatically fixes manual changes.

Example:

Engineer changes replicas manually:

```
kubectl scale deployment app --replicas=1
```

Git says:

```
replicas=5
```

ArgoCD restores:

```
replicas=5
```

---

## Prune

Definition:

Deletes resources removed from Git.

Example:

Resource removed from repository.

ArgoCD removes it from cluster.

---

# ApplicationSets

## Definition

ApplicationSets create multiple ArgoCD applications automatically.

Useful for:

- Multiple clusters
- Multiple environments
- Many applications

---

Example:

Before:

```
Application A

Application B

Application C
```

Manually created.

---

After:

```
ApplicationSet

 |

Creates Applications Automatically
```

---

# Multi-Cluster GitOps

## Problem

Large organizations operate:

- Multiple environments
- Multiple regions
- Multiple Kubernetes clusters

---

# Multi-Cluster Pattern

```
Git Repository

 |

ArgoCD

 |

 +-- Dev Cluster

 +-- Staging Cluster

 +-- Production Cluster
```

---

# Benefits

- Centralized deployment model
- Consistent configurations
- Easier auditing
- Safer releases

---

# Configuration Management Approaches

Common approaches:

- Helm
- Kustomize
- Jsonnet

---

# Helm with ArgoCD

Flow:

```
Git

 |

Helm Chart

 |

ArgoCD

 |

Kubernetes
```

Benefits:

- Reusable templates
- Environment values

---

# Kustomize with ArgoCD

## Definition

Kustomize modifies Kubernetes YAML without templates.

Example:

Base:

```
deployment.yaml
```

Overlays:

```
dev

production
```

---

# Deployment Promotion

Common pattern:

```
Development

    |

    v

Staging

    |

    v

Production
```

Promotion happens through Git changes.

---

# Progressive Delivery

## Definition

Progressive delivery gradually introduces changes while monitoring health.

Examples:

- Canary deployment
- Blue/Green deployment

---

# Canary with GitOps

Flow:

```
New Version

 |

10% Traffic

 |

Monitor

 |

Increase Traffic

 |

100%
```

---

# Rollback Strategy

GitOps rollback is simple:

Revert Git commit.

Example:

Bad deployment:

```
Commit B
```

Rollback:

```
git revert B
```

ArgoCD restores previous state.

---

# CI/CD and GitOps Integration

Recommended architecture:

```
Developer

 |

Source Code

 |

CI Pipeline

 |

Build Image

 |

Container Registry

 |

Update Deployment Manifest

 |

Git Commit

 |

ArgoCD

 |

Kubernetes
```

---

# CI Responsibilities

CI should:

- Build artifacts
- Run tests
- Scan images
- Publish images

---

# GitOps Responsibilities

GitOps should:

- Deploy applications
- Maintain desired state
- Reconcile drift

---

# Production Incident Scenarios

---

# Incident: ArgoCD Application OutOfSync

## Symptoms

Application shows:

```
OutOfSync
```

---

## Investigation

Check:

1. Git changes
2. Cluster differences
3. Resource ownership

Commands:

```
argocd app diff application-name
```

---

## Possible Causes

### Manual Cluster Change

Example:

Engineer changed deployment directly.

Fix:

Restore Git state.

---

### Wrong Repository Configuration

Example:

Incorrect branch or path.

Fix:

Validate Application configuration.

---

### Permission Failure

Example:

ArgoCD cannot update resources.

Fix:

Review RBAC permissions.

---

# Incident: Deployment Stuck Progressing

## Symptoms

Application:

```
Progressing
```

for long time.

---

## Investigation

Check:

```
kubectl get pods

kubectl describe deployment app
```

---

Possible causes:

- Image pull failure
- Readiness failure
- Resource shortage
- Configuration error

---

# Incident: ArgoCD Cannot Sync

## Possible Causes

| Cause | Check |
|---|---|
| Repository unavailable | Git connectivity |
| Permission issue | RBAC |
| Invalid YAML | Manifest validation |
| Cluster unavailable | API server |
| Resource conflict | Existing objects |

---

# GitOps Security Best Practices

Use:

- Repository protection
- Pull request approvals
- Signed commits
- Secret management
- Least privilege access

Avoid:

- Direct production kubectl access
- Secrets in Git
- Shared credentials

---

# Senior Interview Questions

## Easy

### What problem does GitOps solve?

Answer:

GitOps provides a declarative, version-controlled, automated method of managing deployments while reducing configuration drift.

---

## Medium

### Why should CI pipelines not directly deploy to Kubernetes?

Answer:

Direct deployment requires cluster credentials in CI systems. GitOps separates build and deployment responsibilities, improving security and auditability.

---

## Hard

### How would you manage deployments across hundreds of Kubernetes clusters?

Answer:

"I would use GitOps with centralized patterns, ApplicationSets, environment repositories, automated promotion workflows, and policy enforcement. The platform should make deployment consistent while allowing teams controlled autonomy."

---

## Staff Level

### How do you design a GitOps platform for enterprise scale?

Answer:

"I would treat GitOps as a platform capability.

The design would include:

- Standard application templates
- Multi-cluster management
- Automated reconciliation
- Policy enforcement
- Deployment observability
- Self-service onboarding
- Disaster recovery procedures"

---

# Common Mistakes

## Mistake 1

Using GitOps only as a deployment tool.

Reality:

GitOps is an operational model.

---

## Mistake 2

Allowing manual production changes.

Problem:

Creates drift.

---

## Mistake 3

Putting secrets directly into Git.

Problem:

Security exposure.

---

# Key Takeaways

A Senior Platform Engineer should understand:

- GitOps principles
- ArgoCD architecture
- Application lifecycle
- Multi-cluster deployment
- Progressive delivery
- Rollbacks
- Deployment troubleshooting

The goal:

Create a reliable deployment platform where engineers can ship safely at scale.

---
