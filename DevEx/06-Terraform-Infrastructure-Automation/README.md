# Terraform Infrastructure Automation

## Introduction

Infrastructure at enterprise scale cannot be managed manually.

Platform teams use Infrastructure as Code (IaC) to create:

- Repeatable environments
- Version-controlled infrastructure
- Automated provisioning
- Governance
- Faster developer onboarding

Terraform is one of the most widely used IaC tools.

---

# Infrastructure as Code (IaC)

## Definition

Infrastructure as Code means managing infrastructure through configuration files instead of manual cloud console changes.

Example:

Manual:

```
Engineer

 |

AWS Console

 |

Create VPC
```

IaC:

```
Terraform Code

 |

Terraform Apply

 |

AWS Resources
```

---

# Why IaC Matters

Without IaC:

- Manual errors
- Configuration drift
- Poor auditability
- Slow provisioning

With IaC:

- Reviewable changes
- Repeatability
- Automation
- Disaster recovery

---

# Terraform

## Definition

Terraform is a declarative Infrastructure as Code tool.

You define:

"What should exist"

Terraform determines:

"How to create it"

---

# Terraform Architecture

```
Terraform Code

      |

      v

Terraform Engine

      |

      v

Cloud Provider API

      |

      v

Infrastructure
```

---

# Terraform Components

## Provider

## Definition

A provider allows Terraform to interact with external platforms.

Examples:

- AWS
- Kubernetes
- GitHub
- Azure

---

Example:

```
Terraform

 |

AWS Provider

 |

AWS API
```

---

## Resource

## Definition

A resource represents infrastructure that Terraform manages.

Examples:

- EC2 instance
- VPC
- IAM role
- Kubernetes namespace

---

## Variable

## Definition

Variables allow reusable configuration.

Example:

Instead of:

```
instance_type = "t3.large"
```

Use:

```
instance_type = var.instance_type
```

---

## Output

## Definition

Outputs expose useful information after deployment.

Examples:

- IP address
- Load balancer URL
- Resource IDs

---

# Terraform Workflow

## Step 1: Write Code

Example:

```
main.tf
```

---

## Step 2: Initialize

Command:

```
terraform init
```

Purpose:

Downloads providers and initializes backend.

---

## Step 3: Validate

Command:

```
terraform validate
```

Purpose:

Checks syntax and configuration correctness.

---

## Step 4: Plan

Command:

```
terraform plan
```

Purpose:

Shows expected changes before applying.

Example:

```
+ Create VPC

+ Create Subnet

~ Update Security Group
```

---

## Step 5: Apply

Command:

```
terraform apply
```

Purpose:

Creates or modifies infrastructure.

---

# Terraform State

## Definition

Terraform state tracks the relationship between configuration and real infrastructure.

Example:

Terraform knows:

```
Code:

Create VPC


State:

VPC exists with ID xyz
```

---

# Why State Matters

Terraform compares:

Desired:

```
Terraform Code
```

Current:

```
Terraform State
```

Actual:

```
Cloud Environment
```

---

# State Problems

## Missing State

Symptoms:

Terraform wants to recreate existing resources.

---

## Corrupted State

Symptoms:

Unexpected changes.

---

## State Locking Failure

Symptoms:

Multiple engineers modifying infrastructure simultaneously.

---

# Remote State

## Definition

Remote state stores Terraform state in shared storage.

Common AWS pattern:

```
Terraform

 |

S3 Bucket

 |

State File
```

With locking:

```
DynamoDB Lock Table
```

---

# Why Remote State?

Benefits:

- Team collaboration
- State backup
- Locking
- Security

---

# Terraform Modules

## Definition

A Terraform module is reusable infrastructure code.

Example:

Instead of every team creating:

```
VPC

Subnets

Routes

Security Groups
```

Create:

```
Network Module
```

---

# Module Example

```
modules/

 |

network/

 |

eks/

 |

iam/
```

---

# Enterprise Terraform Structure

Example:

```
Infrastructure Repo

 |

 +-- modules

 |

 +-- environments

       |

       +-- dev

       |

       +-- prod
```

---

# Terraform Best Practices

## Use Modules

Avoid duplicate infrastructure code.

---

## Use Remote State

Avoid local state files.

---

## Review Changes

Use:

- Pull requests
- terraform plan output
- Approvals

---

## Automate Validation

Pipeline should run:

```
terraform fmt

terraform validate

terraform plan
```

---

# Terraform and CI/CD

Recommended flow:

```
Developer

 |

Pull Request

 |

Terraform Plan

 |

Review

 |

Approval

 |

Terraform Apply
```

---

# Terraform Security Practices

Avoid:

```
Passwords in Terraform Files
```

Use:

- Secret managers
- IAM roles
- Environment variables

---

# Terraform Drift

## Definition

Drift occurs when infrastructure changes outside Terraform.

Example:

Terraform:

```
Instance Type:

t3.medium
```

Engineer manually changes:

```
t3.large
```

Now:

```
Code != Infrastructure
```

---

# Detecting Drift

Command:

```
terraform plan
```

Terraform shows unexpected differences.

---

# Preventing Drift

Use:

- Restricted console access
- GitOps workflows
- Automated compliance checks

---

# Terraform with Kubernetes

Terraform can manage:

- EKS clusters
- Node groups
- IAM integration
- Kubernetes namespaces
- Applications

---

# Terraform vs Helm

| Terraform | Helm |
|-|-|
| Infrastructure | Kubernetes applications |
| AWS resources | Pods/services |
| VPC/EKS/IAM | Deployments |

---

# Terraform Production Scenario

# Incident: Terraform Wants to Destroy Production Resources

## Symptoms

Terraform plan shows:

```
Destroy:

Production Database
```

---

## Immediate Actions

Do not apply.

Investigate:

1. State correctness
2. Recent code changes
3. Resource imports
4. Provider changes

---

# Possible Causes

## State Loss

Problem:

Terraform no longer knows resource ownership.

---

## Resource Renamed

Example:

Before:

```
aws_instance.app
```

After:

```
aws_instance.application
```

Terraform thinks:

Delete old.

Create new.

---

## Manual Changes

Infrastructure differs from state.

---

# Terraform Import

## Definition

Import connects existing infrastructure with Terraform management.

Example:

Existing AWS resource:

```
VPC
```

Import:

```
terraform import
```

---

# Terraform Interview Questions

## Easy

### What problem does Terraform solve?

Answer:

Terraform provides repeatable, version-controlled infrastructure management using declarative configuration.

---

## Medium

### Explain Terraform state.

Answer:

Terraform state maps configuration to real resources. It allows Terraform to understand what exists and what changes are required.

---

## Hard

### How would you manage Terraform across hundreds of AWS accounts?

Answer:

"I would use reusable modules, remote state, CI validation, account-specific environments, policy controls, and automated workflows."

---

## Staff Level

### How do you design an enterprise infrastructure platform?

Answer:

"I would create a paved road approach:

- Approved Terraform modules
- Self-service provisioning
- Security guardrails
- Automated validation
- Cost controls
- Ownership standards

The platform should enable teams without allowing unmanaged infrastructure growth."

---

# Common Terraform Mistakes

## Mistake 1

One huge Terraform file.

Problem:

Hard to maintain.

---

## Mistake 2

Sharing state incorrectly.

Problem:

Concurrent changes create failures.

---

## Mistake 3

No plan review.

Problem:

Unexpected production changes.

---

# Key Takeaways

Senior Platform Engineers should understand:

- Terraform architecture
- State management
- Modules
- Remote state
- Drift handling
- CI/CD integration
- Enterprise governance

Terraform is not just provisioning.

It is a platform capability for creating safe, scalable infrastructure operations.

---
