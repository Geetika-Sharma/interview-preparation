# 11 - CI/CD + GitOps Failure Systems (Senior SRE Interview Guide)

A production-grade deep dive into CI/CD pipelines, GitOps systems, and deployment failures at scale (ArgoCD, Jenkins, Helm, Kubernetes rollouts in large microservice environments).

---

# Table of Contents

- Introduction
- Why CI/CD Becomes a Reliability System
- CI/CD Architecture at Scale
- Jenkins Pipeline Failures
- GitOps (ArgoCD) Failures
- Helm / Manifest Rendering Failures
- Deployment Rollout Failures
- Rollback Failures (Critical Incident Class)
- Version Drift & Configuration Skew
- Deployment-Induced Outages
- Real Production Incident Patterns
- Debugging Flow
- Senior SRE Mental Model
- Interview Answer Template
- Key Takeaways

---

# Introduction

At scale, CI/CD is no longer just a delivery mechanism.

It becomes:

> A **distributed control system for production state**

So when CI/CD breaks:
- production becomes inconsistent
- services partially deploy
- rollback becomes unsafe
- system state drifts silently

---

# Why CI/CD Becomes a Reliability System

Because it directly controls:
- what version runs in production
- how configuration is applied
- how Kubernetes objects evolve

So failures here cause:
- partial deployments
- inconsistent clusters
- hidden version skew

---

# CI/CD Architecture at Scale

Typical pipeline:

    Git Commit
        ↓
    CI Build (Jenkins / GitHub Actions)
        ↓
    Artifact Registry (Nexus / ECR)
        ↓
    CD System (ArgoCD / Spinnaker)
        ↓
    Kubernetes Deployment
        ↓
    Runtime Validation

---

# 1. Jenkins / CI Pipeline Failures

## Definition

Failures during build, test, or artifact creation.

---

## Failure Modes

### 1. Build agent exhaustion
- no available runners

### 2. Dependency resolution failure
- broken package versions

### 3. Test flakiness at scale
- non-deterministic failures

### 4. Artifact upload failure
- registry downtime

---

## Symptoms

- delayed deployments
- pipeline queue buildup
- inconsistent build results

---

## Fix

- scale CI runners
- cache dependencies
- isolate flaky tests

---

# 2. GitOps (ArgoCD) Failures

## Definition

GitOps ensures Kubernetes state matches Git state.

---

## Failure Modes

### 1. Sync drift
- cluster state != git state

### 2. Controller lag
- ArgoCD unable to reconcile changes

### 3. RBAC misconfig
- cannot apply changes

### 4. Repo connectivity issues

---

## Symptoms

- deployments stuck in "OutOfSync"
- partial rollouts
- inconsistent environments

---

## Fix

- force sync reconciliation
- fix RBAC permissions
- stabilize Git connectivity

---

# 3. Helm / Manifest Rendering Failures

## Definition

Kubernetes manifests fail to render or apply correctly.

---

## Failure Modes

### 1. Template errors
- invalid values.yaml

### 2. Schema mismatches
- API version changes

### 3. Environment-specific overrides broken

---

## Symptoms

- deployment fails before rollout
- invalid resource definitions

---

## Fix

- validate templates
- enforce schema validation
- version Helm charts properly

---

# 4. Deployment Rollout Failures

## Definition

Pods deploy but do not successfully replace old version.

---

## Failure Modes

### 1. Readiness probe failures
### 2. Image pull errors
### 3. CrashLoopBackOff during rollout
### 4. Slow rollout due to dependency latency

---

## Symptoms

- partial deployments
- traffic split between versions
- inconsistent behavior

---

## Fix

- rollback deployment
- fix readiness checks
- stabilize dependencies

---

# 5. Rollback Failures (Critical Class)

## Definition

Rollback does not restore stable system state.

---

## Why it happens

- DB schema changes not backward compatible
- stateful services not versioned properly
- config drift

---

## Symptoms

- rollback still broken
- partial recovery only
- cascading instability

---

## Fix

- ensure backward compatibility
- use feature flags
- test rollback paths

---

# 6. Version Drift & Configuration Skew

## Definition

Different services run different versions unexpectedly.

---

## Causes

- partial rollout
- manual hotfixes
- failed sync jobs

---

## Symptoms

- inconsistent API behavior
- debugging confusion
- hard-to-reproduce bugs

---

## Fix

- enforce version pinning
- enforce GitOps reconciliation loops

---

# 7. Deployment-Induced Outages

## Definition

A deployment causes system-wide degradation.

---

## Causes

- bad config rollout
- dependency mismatch
- resource misallocation

---

## Symptoms

- sudden latency spikes
- retry storms
- cascading failures

---

## Fix

- rollback immediately
- isolate new version traffic
- progressive rollout strategy

---

# 8. Real Production Incident Patterns

---

## Pattern 1: “Deployment stuck halfway”

Cause:
- ArgoCD sync failure or RBAC issue

---

## Pattern 2: “New version causes latency spike”

Cause:
- dependency regression or config change

---

## Pattern 3: “Rollback didn’t fix issue”

Cause:
- DB schema incompatibility

---

## Pattern 4: “Random version mismatch across services”

Cause:
- partial rollout + sync drift

---

# 9. Debugging Flow

    Step 1: Check deployment status
        ↓
    Step 2: Check CI pipeline health
        ↓
    Step 3: Check GitOps sync status
        ↓
    Step 4: Validate Kubernetes rollout state
        ↓
    Step 5: Check version consistency across services
        ↓
    Step 6: Identify deployment drift or failure point

---

# 10. Senior SRE Mental Model

Junior:
- “Deployment failed”

Mid-level:
- “Check pipeline logs”

Senior:
- “Is this a CI failure, GitOps drift, or rollout instability?”

Principal:
- “Is the deployment system itself introducing inconsistency into production state?”

---

# 11. Interview Answer Template

When diagnosing CI/CD or GitOps-related failures, I first determine whether the issue originates in the build pipeline, artifact registry, deployment controller, or Kubernetes rollout process. I check CI pipeline health for build or test failures, then validate GitOps synchronization status to ensure cluster state matches desired state. I also inspect rollout progress to detect partial deployments or readiness issues. If inconsistencies are present, I look for version drift or configuration skew across services. I mitigate by rolling back to a known stable state, stopping further deployments, and stabilizing the deployment pipeline, followed by improving rollback safety, version compatibility, and GitOps reconciliation reliability.

---

# 12. Key Takeaways

- CI/CD is a production control system, not just a delivery tool
- GitOps failures cause silent cluster drift
- Rollback safety is as important as deployment safety
- Partial rollouts are a major production risk
- Version skew creates hard-to-debug distributed issues

---

# End of Module 11
