# Senior Platform Engineer Interview Questions

## AWS Infrastructure

This section covers senior-level AWS infrastructure questions focused on production operations, reliability, security, automation, and platform engineering practices.

---

## Q1. What is EC2?

Answer:

Amazon EC2 is AWS's virtual compute service that provides scalable compute capacity.

At a senior engineering level, EC2 should not be treated as manually managed servers. It should be operated as a standardized compute platform using:

- Infrastructure as Code
- Immutable infrastructure
- Automated provisioning
- Monitoring
- Security controls
- Cost governance
- Automated recovery

Production best practices:

Reliability:
- Use Auto Scaling Groups.
- Deploy across multiple Availability Zones.
- Replace unhealthy instances automatically.
- Avoid single-instance dependencies.

Security:
- Prefer IAM roles over access keys.
- Use Systems Manager Session Manager instead of unmanaged SSH access.
- Apply least privilege permissions.
- Enable encryption.
- Regularly patch operating systems.

Operations:
- Centralize logs.
- Monitor application and infrastructure metrics.
- Maintain runbooks.
- Automate common operational tasks.

Cost:
- Right-size instances.
- Use Savings Plans or Reserved Instances where appropriate.
- Remove unused resources.
- Track ownership using resource tagging.

A mature platform treats EC2 as an automated capability delivered to engineering teams, not as individual machines.

---

## Q2. What happens when you launch an EC2 instance?

Answer:

When an EC2 instance is launched, AWS performs several steps:

1. Selects physical capacity based on availability.
2. Uses the selected AMI as the operating system template.
3. Creates and attaches storage volumes.
4. Assigns networking resources.
5. Applies security groups.
6. Starts the operating system.
7. Runs initialization scripts.
8. Starts required services.

Production approach:

Avoid manually configuring servers after launch.

Use:

- Versioned AMIs
- Terraform modules
- Automated bootstrap scripts
- Configuration management
- CI/CD-based infrastructure delivery

The goal is repeatable infrastructure creation with minimal configuration drift.

---

## Q3. What is the difference between an AMI and an EC2 instance?

Answer:

An AMI is a template used to create EC2 instances.

An EC2 instance is the running compute resource created from that template.

An AMI contains:

- Operating system
- Installed software
- Configuration baseline

An EC2 instance provides:

- CPU
- Memory
- Network interfaces
- Runtime environment

Senior practice:

Do not modify production servers manually.

Use:

Application change

->

Build new image

->

Security validation

->

Deploy new instances

->

Remove old instances

This supports immutable infrastructure and improves reliability.

---

## Q4. How do you design production-ready EC2 infrastructure?

Answer:

I design EC2 infrastructure using the AWS Well-Architected Framework.

Reliability:

- Multi-AZ deployment
- Auto Scaling Groups
- Load balancing
- Automated recovery
- Health checks

Security:

- IAM roles
- Least privilege access
- Private subnets
- Security groups
- Encryption
- Vulnerability scanning

Operational Excellence:

- Infrastructure as Code
- Automated deployments
- Monitoring
- Runbooks
- Incident processes

Performance:

- Right instance selection
- Capacity planning
- Application profiling

Cost:

- Right sizing
- Usage monitoring
- Reserved capacity where appropriate
- Removing unused resources

The platform goal is to provide reliable compute capabilities without developers managing infrastructure details.

---

## Q5. An EC2 instance is unreachable. How do you troubleshoot?

Answer:

I troubleshoot from the outside inward.

Step 1: AWS layer

Check:

- Instance state
- System status checks
- Instance status checks
- Security groups
- Network ACLs
- Route tables

Step 2: Network layer

Check:

- Subnet configuration
- Internet gateway
- NAT gateway
- Load balancer health checks
- DNS resolution

Step 3: Access layer

Check:

- SSH configuration
- IAM permissions
- Systems Manager availability
- Key configuration

Step 4: Operating system layer

Check:

- CPU usage
- Memory pressure
- Disk availability
- Network services
- System logs

Senior approach:

Do not immediately restart the instance.

First collect evidence to understand the failure.

---

## Q6. An engineer lost the SSH private key for a production EC2 instance. What do you do?

Answer:

The first question is whether SSH should be required at all.

Preferred solution:

Use AWS Systems Manager Session Manager with IAM-based authentication.

If recovery requires modifying access:

1. Stop the instance if required.
2. Detach the root EBS volume.
3. Attach the volume to a recovery instance.
4. Add a new authorized key.
5. Validate file ownership and permissions.
6. Reattach the volume.
7. Start and verify access.

After recovery:

- Remove dependency on individual SSH keys.
- Move to centralized identity management.
- Audit access.
- Rotate credentials.
- Document recovery procedures.

Senior perspective:

The incident is not only a lost key problem. It is an access management design problem.

---

## Q7. EC2 CPU utilization is 100%. How do you troubleshoot?

Answer:

High CPU is a symptom, not the root cause.

First determine:

- Is traffic increasing?
- Did a deployment happen?
- Is the application inefficient?
- Is the instance undersized?

Investigation:

Check processes:

    top
    htop
    ps aux
    pidstat

Analyze:

- Which process consumes CPU?
- User CPU vs system CPU
- CPU throttling
- Context switching
- I/O wait

Check AWS metrics:

- CPU utilization
- Network traffic
- Load balancer requests
- Auto Scaling activity

Immediate actions:

- Roll back bad deployments.
- Scale horizontally.
- Increase capacity if justified.

Long-term improvements:

- Improve autoscaling signals.
- Add performance testing.
- Optimize application behavior.
- Establish capacity planning.

Senior perspective:

The objective is not minimizing CPU. The objective is reliable performance at optimal cost.

---

## Q8. EC2 memory usage keeps increasing. How do you troubleshoot?

Answer:

Continuous memory growth usually indicates:

- Memory leak
- Incorrect caching
- Application growth
- Misconfiguration

Investigation:

Operating system:

    free -m
    top
    vmstat

Application:

- Heap usage
- Garbage collection behavior
- Cache growth
- Connection handling

Infrastructure:

- Instance sizing
- Scaling behavior
- Workload changes

Long-term improvements:

- Memory dashboards
- Application profiling
- Load testing
- Automated alerts

---

## Q9. EC2 disk space is full. How do you troubleshoot?

Answer:

First identify what consumed the storage.

Check filesystem usage:

    df -h

Find large files:

    du -sh /*

Common causes:

- Application logs
- Temporary files
- Package caches
- Database files
- Container images

Immediate mitigation:

- Rotate logs.
- Remove unnecessary files.
- Extend EBS volume if required.

Prevention:

- Centralized logging.
- Retention policies.
- Disk utilization alerts.
- Automated cleanup.

Senior approach:

Prevent resource exhaustion through monitoring and automation rather than reacting after failure.

---

## Q10. How do you secure an EC2 fleet?

Answer:

I use multiple layers of security.

Identity:

- IAM roles instead of credentials.
- Least privilege permissions.
- MFA for privileged actions.

Network:

- Private subnets.
- Security groups.
- Restricted inbound access.
- VPC endpoints where appropriate.

Host:

- Hardened AMIs.
- Patch automation.
- Vulnerability scanning.
- Endpoint monitoring.

Operations:

- Centralized audit logs.
- AWS CloudTrail.
- Security monitoring.
- Automated compliance checks.

Platform approach:

Security should be built into the golden path instead of relying on individual engineers to remember every control.

---

## Q11. How do you manage thousands of EC2 instances?

Answer:

At scale, manual management does not work.

I would use:

Infrastructure:

- Terraform modules.
- Standardized AMIs.
- Automated provisioning.

Configuration:

- Systems Manager.
- Configuration management.
- Patch automation.

Operations:

- Centralized monitoring.
- Automated remediation.
- Ownership tagging.

Governance:

- Resource standards.
- Policy enforcement.
- Cost allocation.

Success metrics:

- Patch compliance
- Availability
- MTTR
- Infrastructure drift
- Cost per workload

---
---

## Q12. What is the difference between vertical scaling and horizontal scaling?

Answer:

Vertical scaling means increasing resources of an existing instance.

Example:

- More CPU
- More memory
- Larger EC2 instance type

Horizontal scaling means adding more instances.

Example:

- Increasing Auto Scaling Group capacity
- Adding more application servers

Senior platform preference:

For production systems, horizontal scaling is usually preferred because it improves:

- Availability
- Fault tolerance
- Deployment flexibility

Vertical scaling is useful for:

- Databases
- Specialized workloads
- Applications that cannot scale horizontally

A mature platform evaluates scaling based on:

- Traffic patterns
- Cost
- Performance requirements
- Recovery objectives

---

## Q13. What is an Auto Scaling Group and how does it work?

Answer:

An Auto Scaling Group manages the lifecycle of EC2 instances automatically.

It maintains:

- Minimum instance count
- Desired instance count
- Maximum instance count

When capacity changes:

- New instances are created.
- Health checks are performed.
- Unhealthy instances are replaced.
- Traffic is distributed through load balancers.

Best practices:

- Use immutable AMIs.
- Use launch templates.
- Configure health checks.
- Scale based on meaningful metrics.

Avoid scaling only on CPU.

Better signals:

- Request count
- Queue depth
- Application latency
- Custom business metrics

Operational metrics:

- Availability
- Scaling success rate
- Instance replacement frequency
- Deployment success rate

---

## Q14. How do you decide when to scale an application?

Answer:

Scaling decisions should be based on workload behavior, not assumptions.

I evaluate:

Application metrics:

- Request latency
- Error rate
- Throughput

Infrastructure metrics:

- CPU
- Memory
- Network
- Disk

Business metrics:

- User activity
- Transactions
- Queue backlog

Example:

CPU at 90% does not always require scaling.

If:

- Requests are normal
- Latency is healthy
- Errors are low

Then scaling may not be necessary.

A senior engineer connects infrastructure metrics to user impact.

---

## Q15. What is an Elastic Load Balancer?

Answer:

Elastic Load Balancer distributes incoming traffic across multiple targets.

Types:

Application Load Balancer:

Used for:

- HTTP/HTTPS
- Layer 7 routing
- Path-based routing

Network Load Balancer:

Used for:

- High performance
- TCP/UDP workloads
- Low latency requirements

Gateway Load Balancer:

Used for:

- Network appliances
- Security inspection

Production best practices:

- Enable health checks.
- Use multiple Availability Zones.
- Configure connection draining.
- Monitor latency and errors.

Security:

- Integrate with WAF when required.
- Use TLS termination.
- Restrict backend access.

---

## Q16. How does an Application Load Balancer work?

Answer:

ALB operates at Layer 7 and makes routing decisions based on HTTP information.

Flow:

Client request

to

ALB listener

to

Listener rules

to

Target group

to

Healthy EC2 instances

Capabilities:

- Host-based routing
- Path-based routing
- TLS termination
- Health checks
- Traffic distribution

Senior considerations:

- Configure meaningful health checks.
- Avoid routing traffic to unhealthy applications.
- Monitor target response time.
- Use gradual deployments with weighted routing.

---

## Q17. A load balancer shows healthy targets, but users receive errors. What do you check?

Answer:

Healthy targets only confirm the health check endpoint works.

I investigate:

Application:

- Actual API failures
- Dependency failures
- Database latency
- Application logs

Load balancer:

- Listener rules
- Target group configuration
- Timeout settings
- Connection limits

Network:

- Security groups
- Network ACLs
- Routing

Observability:

- Request latency
- Error percentage
- Distributed traces

Senior approach:

Health checks should represent real application health, not only process availability.

---

## Q18. What is the difference between Security Groups and Network ACLs?

Answer:

Security Groups:

- Instance-level firewall.
- Stateful.
- Allow rules only.
- Return traffic is automatically allowed.

Network ACLs:

- Subnet-level firewall.
- Stateless.
- Allow and deny rules.
- Requires inbound and outbound rules.

Typical production usage:

Security Groups:

- Application access control.
- Service-to-service communication.

Network ACLs:

- Additional network boundary protection.

Best practice:

Use Security Groups as the primary control and NACLs as an additional layer.

---

## Q19. How do you design AWS networking for a production platform?

Answer:

A production AWS network should provide:

- Isolation
- Security
- Scalability
- Observability

Typical design:

Public subnet:

- Load balancers
- Internet-facing components

Private subnet:

- Applications
- Workers
- Internal services

Restricted subnet:

- Databases
- Sensitive workloads

Best practices:

- Multiple Availability Zones.
- Controlled routing.
- VPC endpoints where appropriate.
- Centralized network logging.
- Least privilege security groups.

---

## Q20. What is the difference between public and private subnets?

Answer:

A public subnet has a route to an Internet Gateway.

A private subnet does not directly expose resources to the internet.

Production pattern:

Public:

- Load balancers
- Bastion alternatives

Private:

- Application servers
- Containers
- Internal services

Sensitive:

- Databases

Security principle:

Only expose components that must receive external traffic.

---

## Q21. How does NAT Gateway work?

Answer:

NAT Gateway allows resources in private subnets to access the internet without allowing inbound internet access.

Example:

Private EC2 instance:

needs

software updates

through

NAT Gateway

through

Internet Gateway

Best practices:

- Deploy NAT Gateway per Availability Zone for resilience.
- Monitor NAT costs.
- Use VPC endpoints where possible.

Cost optimization:

High NAT Gateway costs often indicate workloads that should use:

- S3 Gateway Endpoint
- PrivateLink
- Internal AWS services

---

## Q22. What is Amazon EBS?

Answer:

EBS is AWS block storage attached to EC2 instances.

It provides:

- Persistent storage
- Snapshots
- Different performance tiers

Common types:

- General purpose SSD
- Provisioned IOPS SSD
- Throughput optimized storage

Production considerations:

Reliability:

- Regular snapshots.
- Multi-AZ backup strategy.

Performance:

- Monitor latency.
- Monitor IOPS.
- Monitor throughput.

Cost:

- Right-size volumes.
- Delete unused snapshots.

---

## Q23. An EC2 instance has high disk latency. How do you troubleshoot?

Answer:

I investigate from application to infrastructure.

Application:

- Increased writes
- Database activity
- Logging changes

Operating system:

Commands:

    iostat
    vmstat
    df -h

Check:

- Disk utilization
- I/O wait
- Queue depth

AWS:

Check:

- EBS volume metrics
- IOPS
- Throughput
- Burst balance

Mitigation:

- Optimize workload.
- Increase EBS performance.
- Move workload architecture if required.

---

## Q24. What is the difference between EBS and EFS?

Answer:

EBS:

- Block storage.
- Attached to instances.
- Usually one instance at a time.
- Good for databases and OS disks.

EFS:

- Managed file system.
- Multiple instances can access simultaneously.
- Good for shared files.

Decision:

Use EBS when applications need high-performance block storage.

Use EFS when multiple services require shared file access.

---

## Q25. How do you perform disaster recovery for EC2 workloads?

Answer:

Disaster recovery strategy depends on business requirements.

Key factors:

RTO:

How quickly must service recover?

RPO:

How much data loss is acceptable?

Strategies:

Backup and restore:

- Lowest cost.
- Slowest recovery.

Pilot light:

- Minimal infrastructure running.

Warm standby:

- Reduced production environment.

Multi-site:

- Full duplicate environment.

Best practices:

- Automate recovery.
- Test regularly.
- Document runbooks.
- Measure recovery time.

A backup without restore testing is not a recovery strategy.

---

## Q26. How do you reduce AWS infrastructure cost?

Answer:

Cost optimization should be continuous.

Approach:

Visibility:

- Resource tagging.
- Cost allocation.
- Usage dashboards.

Optimization:

- Right-size instances.
- Remove unused resources.
- Use Auto Scaling.
- Use Savings Plans.
- Use Spot Instances where appropriate.

Architecture:

- Reduce unnecessary data transfer.
- Use managed services when cheaper operationally.
- Optimize storage tiers.

Senior perspective:

The goal is not simply reducing cost.

The goal is maximizing business value per infrastructure dollar.

---

## Q27. How do you manage multiple AWS accounts?

Answer:

A mature organization uses AWS multi-account architecture.

Typical separation:

- Production
- Development
- Security
- Networking
- Shared services

Governance:

- AWS Organizations.
- Service Control Policies.
- Centralized logging.
- Identity management.
- Standard Terraform modules.

Benefits:

- Isolation.
- Security boundaries.
- Cost visibility.
- Easier compliance.

---

## Q28. How do you implement AWS security governance?

Answer:

I use preventive and detective controls.

Preventive:

- IAM policies.
- Service Control Policies.
- Infrastructure policies.
- Approved templates.

Detective:

- CloudTrail.
- Config rules.
- Security Hub.
- Vulnerability scanning.

Platform approach:

Developers should receive secure defaults through golden paths instead of manually implementing security controls.

---

## Q29. What AWS metrics do you monitor for production EC2 workloads?

Answer:

Infrastructure metrics:

- CPU utilization
- Memory utilization
- Disk usage
- Network throughput

Application metrics:

- Request latency
- Error rate
- Throughput

Reliability metrics:

- Availability
- MTTR
- Incident frequency
- Change failure rate

Cost metrics:

- Cost per service
- Resource utilization
- Idle capacity

Senior monitoring focuses on customer impact, not just infrastructure health.

---

## Q30. How do you safely make infrastructure changes in production?

Answer:

I use controlled change management.

Process:

1. Define desired change.
2. Review through code.
3. Validate automatically.
4. Test in lower environments.
5. Deploy gradually.
6. Monitor impact.
7. Roll back if required.

Practices:

- Terraform plan reviews.
- CI validation.
- Automated testing.
- Canary changes.

This improves DORA metrics:

- Lower change failure rate.
- Faster recovery.
- Higher deployment frequency.

---
---

## Q31. What is IAM and why is it important?

Answer:

AWS Identity and Access Management (IAM) controls who can access AWS resources and what actions they can perform.

A senior approach treats IAM as a core security boundary.

Best practices:

- Prefer IAM roles over long-lived access keys.
- Follow least privilege.
- Separate human access from workload access.
- Use temporary credentials.
- Review permissions regularly.

For production platforms:

Developers should consume approved roles through:

- SSO
- Identity providers
- Permission boundaries
- Automated provisioning

Avoid:

- Shared accounts
- Static credentials
- Administrator access for daily work

---

## Q32. What is the difference between IAM User, Role, and Policy?

Answer:

IAM User:

Represents a specific identity.

Example:

A human engineer.

IAM Role:

Provides temporary permissions assumed by users or services.

Example:

EC2 instance accessing S3.

IAM Policy:

Defines allowed or denied actions.

Example:

Allow reading objects from a specific S3 bucket.

Production preference:

Use roles wherever possible.

Example:

Application:

EC2 instance

assumes

IAM Role

gets

temporary S3 permissions

instead of storing:

AWS access key

and

secret key

---

## Q33. How do you design IAM permissions for hundreds of engineers?

Answer:

I avoid manually managing individual permissions.

Use:

- Identity provider integration
- Groups
- Roles
- Permission boundaries
- Automated provisioning

Example model:

Developer:

Read-only production access

Platform Engineer:

Infrastructure management access

Security Team:

Audit access

Break-glass:

Emergency elevated access

Controls:

- MFA
- Approval workflows
- Access reviews
- Audit logging

The goal is enabling engineers without creating security risk.

---

## Q34. An application has hardcoded AWS access keys. What do you do?

Answer:

Immediate actions:

1. Rotate the exposed credentials.
2. Identify where they are used.
3. Review CloudTrail activity.
4. Remove the credentials from code.

Long-term fix:

Replace credentials with:

- IAM roles
- Instance profiles
- Workload identity
- Secret management systems

Prevention:

- Secret scanning in CI/CD.
- Developer education.
- Automated policy checks.

Senior perspective:

The solution is not only removing one key. It is eliminating the pattern.

---

## Q35. How do you troubleshoot an IAM AccessDenied error?

Answer:

I follow a structured approach.

Check:

1. Identity:

Who is making the request?

2. Permissions:

Does the policy allow the action?

3. Resource:

Is the resource ARN correct?

4. Conditions:

Are policy conditions blocking access?

5. Organization controls:

Are Service Control Policies denying access?

6. Explicit deny:

Remember:

Explicit deny always wins.

Tools:

- IAM Policy Simulator
- CloudTrail
- Access Analyzer

---

## Q36. What is AWS CloudTrail?

Answer:

CloudTrail records AWS API activity.

It answers:

- Who performed an action?
- What changed?
- When did it happen?
- From where?

Production uses:

Security:

- Investigate unauthorized changes.

Operations:

- Troubleshoot infrastructure changes.

Compliance:

- Provide audit evidence.

Best practices:

- Centralize logs.
- Protect log storage.
- Enable across accounts.
- Monitor sensitive actions.

---

## Q37. How do you investigate an unexpected AWS resource change?

Answer:

I start with audit evidence.

Process:

1. Check CloudTrail.
2. Identify the API call.
3. Identify the user or role.
4. Review related changes.
5. Determine if it was expected.

Then:

- Roll back if required.
- Improve controls.
- Update automation.

Prevention:

- Infrastructure as Code.
- Restricted manual changes.
- Drift detection.

---

## Q38. What is Route53?

Answer:

Route53 is AWS's managed DNS service.

It provides:

- Domain registration
- DNS resolution
- Health checks
- Traffic routing

Routing policies include:

- Simple
- Weighted
- Latency-based
- Failover
- Geolocation

Production usage:

Commonly used for:

- Service discovery
- Application routing
- Disaster recovery

---

## Q39. Users cannot access an application. How do you troubleshoot DNS?

Answer:

I follow the DNS resolution path.

Check:

1. Domain registration.

2. DNS records:

- A record
- CNAME
- Alias

3. Route53 health checks.

4. Load balancer configuration.

5. Application availability.

Commands:

    nslookup
    dig

Also verify:

- TTL behavior
- Recent DNS changes
- Propagation delay

Senior approach:

DNS failures can look like application failures, so validate each layer.

---

## Q40. How do you design DNS for highly available applications?

Answer:

Use DNS as part of reliability design.

Approaches:

- Health checks.
- Failover routing.
- Multi-region routing.
- Weighted deployments.

Example:

Production:

Primary region

+

Secondary region

If health checks fail:

Traffic shifts automatically.

Considerations:

- TTL values.
- Recovery time.
- Application state.

---

## Q41. What is Amazon S3?

Answer:

S3 is AWS object storage designed for high durability and scalability.

Common uses:

- Application assets
- Backups
- Logs
- Data storage
- Artifacts

Production considerations:

Security:

- Block public access.
- Bucket policies.
- Encryption.
- Access logging.

Reliability:

- Versioning.
- Lifecycle policies.
- Replication.

Cost:

- Storage classes.
- Lifecycle movement.
- Cleanup policies.

---

## Q42. How do you secure an S3 bucket?

Answer:

Security layers:

Access control:

- Block public access.
- Least privilege policies.
- IAM roles.

Data protection:

- Encryption at rest.
- Encryption in transit.
- Key management.

Monitoring:

- CloudTrail.
- Access logs.
- Security alerts.

Governance:

- Automated policy checks.
- Infrastructure as Code.

Common mistake:

Making buckets public for convenience.

---

## Q43. An application cannot upload files to S3. How do you troubleshoot?

Answer:

Check layers:

Application:

- Correct bucket name.
- Correct region.
- Request errors.

IAM:

- Role permissions.
- Bucket policy.
- KMS permissions.

Network:

- VPC endpoint.
- Connectivity.

AWS:

- S3 availability.
- Service limits.

Evidence:

- Application logs.
- CloudTrail.
- AWS error messages.

---

## Q44. How do you manage Terraform for AWS at enterprise scale?

Answer:

I treat Terraform as a platform capability.

Standards:

- Reusable modules.
- Version control.
- Code review.
- Automated validation.

State management:

- Remote backend.
- State locking.
- Controlled access.

Governance:

- Policy checks.
- Security scanning.
- Cost checks.

Developer experience:

Provide golden path modules:

Example:

Approved:

"Create production service infrastructure"

Instead of:

"Write raw AWS resources manually"

---

## Q45. What is infrastructure drift?

Answer:

Infrastructure drift occurs when actual cloud resources differ from the desired Infrastructure as Code state.

Example:

Terraform defines:

EC2 instance type: m6.large

Someone manually changes:

EC2 instance type: m6.xlarge

Now:

Code != Environment

Detection:

- Terraform plan.
- Cloud configuration tools.
- Auditing.

Prevention:

- Reduce manual changes.
- Enforce IaC.
- Automate validation.

---

## Q46. Terraform plan shows unexpected resource deletion. What do you do?

Answer:

I never blindly apply.

Steps:

1. Stop the deployment.
2. Review Terraform plan.
3. Identify why Terraform thinks deletion is required.
4. Check state.
5. Review recent changes.

Possible causes:

- Resource renamed.
- Missing import.
- State issue.
- Provider behavior change.
- Manual drift.

Resolution:

- Correct configuration.
- Import resources if required.
- Validate again.

---

## Q47. How do you handle Terraform state securely?

Answer:

Terraform state contains sensitive information.

Best practices:

- Remote backend.
- Encryption.
- Access control.
- State locking.
- Audit access.

Avoid:

- Local state files.
- Shared state without controls.

Operational practices:

- Backup state.
- Restrict permissions.
- Review changes.

---

## Q48. How do you design a multi-account AWS platform?

Answer:

A mature AWS environment separates responsibilities.

Typical accounts:

- Production workloads
- Non-production workloads
- Security
- Logging
- Networking
- Shared services

Controls:

- AWS Organizations.
- Service Control Policies.
- Central identity.
- Central monitoring.

Benefits:

- Better isolation.
- Better governance.
- Better cost visibility.
- Reduced blast radius.

---

## Q49. What is AWS Well-Architected Framework?

Answer:

AWS Well-Architected provides principles for designing cloud workloads.

Six pillars:

1. Operational Excellence

Focus:

- Automation
- Monitoring
- Continuous improvement

2. Security

Focus:

- Identity
- Protection
- Detection

3. Reliability

Focus:

- Recovery
- Fault tolerance
- Testing

4. Performance Efficiency

Focus:

- Right resources
- Optimization

5. Cost Optimization

Focus:

- Efficient spending

6. Sustainability

Focus:

- Resource efficiency

Senior engineers use these principles during architecture decisions.

---

## Q50. How do you measure infrastructure platform success?

Answer:

I measure both technical and developer impact.

Reliability metrics:

- Availability
- MTTR
- Incident frequency
- Change failure rate

DORA metrics:

- Deployment frequency
- Lead time for changes
- Change failure rate
- Time to restore service

Platform metrics:

- Self-service adoption
- Developer onboarding time
- Pipeline success rate
- Build duration
- Infrastructure provisioning time

Cost metrics:

- Cost per application
- Resource utilization
- Waste reduction

A successful platform improves engineering velocity without sacrificing reliability or security.

---

---

## Q51. What is the difference between ECS and EKS?

Answer:

ECS and EKS are AWS container orchestration services.

ECS:

- AWS-native container orchestration.
- Simpler operational model.
- Tight AWS integration.

EKS:

- Managed Kubernetes service.
- Kubernetes-compatible.
- Better for organizations standardizing on Kubernetes.

Decision factors:

Use ECS when:

- AWS-only environment.
- Simpler container operations are preferred.
- Kubernetes ecosystem is not required.

Use EKS when:

- Multiple teams need Kubernetes.
- Platform engineering is required.
- Teams use Kubernetes tooling and patterns.

Senior consideration:

The choice is not only technical. Consider:

- Team expertise.
- Operational overhead.
- Developer experience.
- Long-term platform strategy.

---

## Q52. Explain EKS architecture.

Answer:

EKS provides managed Kubernetes control plane with AWS-managed components.

Major components:

Control plane:

- Kubernetes API server.
- etcd.
- Scheduler.
- Controller manager.

Data plane:

- Worker nodes.
- Managed node groups.
- Fargate profiles.

AWS integrations:

- VPC CNI.
- IAM integration.
- Load balancers.
- EBS storage.

Production considerations:

Reliability:

- Multiple Availability Zones.
- Managed node groups.
- Cluster autoscaling.

Security:

- IAM roles.
- Network policies.
- Pod security controls.
- Secrets management.

Operations:

- Central logging.
- Metrics.
- Automated upgrades.

---

## Q53. How do you design an EKS platform for hundreds of developers?

Answer:

I would build a platform, not just a Kubernetes cluster.

Core capabilities:

Developer experience:

- Application templates.
- Standard Helm charts.
- GitOps workflows.
- Self-service deployments.

Security:

- Namespace isolation.
- RBAC.
- Admission policies.
- Image scanning.

Operations:

- Central observability.
- Standard dashboards.
- Automated remediation.

Governance:

- Resource quotas.
- Cost visibility.
- Ownership tagging.

Golden path:

Developer creates service

->

Repository template

->

CI pipeline

->

Security checks

->

GitOps deployment

->

Monitoring enabled

---

## Q54. How do you manage Kubernetes costs in EKS?

Answer:

Kubernetes cost optimization requires visibility first.

Areas to optimize:

Compute:

- Right-size nodes.
- Use autoscaling.
- Remove unused workloads.

Resources:

- Correct CPU/memory requests.
- Remove over-provisioning.
- Use workload metrics.

Architecture:

- Use appropriate instance types.
- Use Spot Instances where suitable.

Governance:

- Namespace cost allocation.
- Resource quotas.
- Showback/chargeback models.

Platform metric:

Measure cost per application or service rather than only cluster cost.

---

## Q55. What happens when a Kubernetes node fails in EKS?

Answer:

The impact depends on workload design.

Flow:

1. Node becomes unhealthy.
2. Kubernetes detects node failure.
3. Pods are marked unavailable.
4. Controllers attempt replacement.
5. Scheduler places replacement pods.

Recovery depends on:

- Replica count.
- Pod disruption budgets.
- Available capacity.
- Autoscaling configuration.

Prevention:

- Multiple nodes across Availability Zones.
- Proper resource requests.
- Health monitoring.
- Automated node replacement.

---

## Q56. How do you upgrade an EKS cluster safely?

Answer:

I treat Kubernetes upgrades as production changes.

Process:

1. Review Kubernetes version compatibility.
2. Test in non-production.
3. Validate workloads and add-ons.
4. Upgrade control plane.
5. Upgrade worker nodes.
6. Monitor applications.
7. Roll back if required.

Check compatibility:

- CNI version.
- Ingress controllers.
- Storage drivers.
- Helm charts.
- Custom resources.

Best practice:

Automate upgrades through tested processes instead of manual changes.

---

## Q57. A Kubernetes application cannot reach an AWS service. How do you troubleshoot?

Answer:

I troubleshoot layer by layer.

Application:

- Verify endpoint.
- Check application logs.

Kubernetes:

- Pod status.
- DNS resolution.
- Service configuration.

Networking:

- VPC routing.
- Security groups.
- Network policies.
- CNI configuration.

AWS:

- IAM permissions.
- Service availability.
- Endpoint configuration.

Security:

Check whether workload identity is configured correctly.

---

## Q58. What is AWS VPC?

Answer:

A VPC is an isolated virtual network in AWS.

It provides:

- IP address ranges.
- Subnets.
- Routing.
- Network security controls.

Production design:

Separate:

- Public resources.
- Private workloads.
- Sensitive systems.

Use:

- Multiple Availability Zones.
- Controlled routing.
- Network logging.
- Security groups.

A VPC is the foundation of AWS security boundaries.

---

## Q59. How do you troubleshoot AWS network connectivity issues?

Answer:

I use a layered approach.

Layer 1:

DNS:

- Name resolution.
- Route53 records.

Layer 2:

Network:

- Routes.
- Subnets.
- NAT gateways.
- Internet gateways.

Layer 3:

Security:

- Security groups.
- NACLs.

Layer 4:

Application:

- Ports.
- Services.
- Dependencies.

Evidence:

- VPC Flow Logs.
- Application logs.
- Network tools.

Senior approach:

Do not change network settings immediately. First identify where traffic stops.

---

## Q60. What are VPC Flow Logs?

Answer:

VPC Flow Logs capture network traffic metadata.

They help answer:

- Who communicated?
- Which IPs were involved?
- Was traffic accepted or rejected?

Uses:

- Troubleshooting connectivity.
- Security investigations.
- Network visibility.

Best practices:

- Centralize logs.
- Protect access.
- Integrate with monitoring.

---

## Q61. A production service has increased latency but no errors. How do you investigate?

Answer:

No errors does not mean no problem.

I investigate:

Application:

- Request latency.
- Dependency latency.
- Database response time.

Infrastructure:

- CPU.
- Memory.
- Network.
- Disk.

Distributed systems:

- External services.
- Queue delays.
- Connection pools.

Observability:

- Metrics.
- Logs.
- Traces.

Senior approach:

Start with user impact and follow the request path.

---

## Q62. How do you design production monitoring for AWS workloads?

Answer:

Monitoring should answer:

"Is the customer impacted?"

I use:

Metrics:

- Availability.
- Latency.
- Error rate.
- Saturation.

Logs:

- Application events.
- Security events.
- Infrastructure events.

Tracing:

- Dependency relationships.
- Slow requests.

Alerting:

Good alerts:

- Actionable.
- Customer-focused.
- Low noise.

Avoid:

- Alerting every infrastructure metric threshold.

---

## Q63. What is the difference between monitoring and observability?

Answer:

Monitoring tells you whether known problems are occurring.

Example:

CPU is high.

Observability helps explain unknown problems.

Example:

Why is checkout latency increasing?

Observability uses:

- Metrics.
- Logs.
- Traces.

Senior platforms invest in observability because production failures are often unexpected.

---

## Q64. How do you handle an AWS regional outage?

Answer:

I first determine business impact.

Questions:

- Which services are affected?
- Which applications depend on the region?
- What recovery options exist?

Recovery strategies:

- Multi-AZ handles availability zone failures.
- Multi-region handles regional failures.

Actions:

- Activate disaster recovery plan.
- Fail traffic over if available.
- Communicate status.
- Track recovery progress.

After recovery:

- Perform postmortem.
- Identify gaps.
- Improve resilience.

---

## Q65. How do you design for AWS Availability Zone failures?

Answer:

AZ failure should be treated as expected.

Design:

- Deploy across multiple AZs.
- Use load balancing.
- Replicate data appropriately.
- Automate failover.

Applications should:

- Avoid single-AZ dependencies.
- Handle instance replacement.
- Maintain graceful degradation.

Test:

- Failure simulations.
- Disaster recovery exercises.

---

## Q66. How do you implement infrastructure standards across AWS?

Answer:

I create paved roads.

Standards include:

Infrastructure:

- Approved Terraform modules.
- Naming conventions.
- Required tags.

Security:

- IAM standards.
- Encryption defaults.
- Network controls.

Operations:

- Monitoring requirements.
- Backup standards.
- Logging requirements.

Developer experience:

Provide reusable templates instead of documentation alone.

---

## Q67. What are AWS tags and why are they important?

Answer:

Tags provide metadata about resources.

Common tags:

- Application.
- Owner.
- Environment.
- Cost center.
- Managed by.
- Data classification.

Uses:

Operations:

- Ownership identification.

Cost:

- Chargeback/showback.

Security:

- Policy enforcement.

Governance:

- Automated compliance checks.

---

## Q68. How do you approach cloud cost optimization?

Answer:

I follow a continuous optimization cycle.

1. Visibility:

Understand spending.

2. Ownership:

Know which team owns resources.

3. Optimization:

- Right-size resources.
- Remove waste.
- Improve architecture.

4. Automation:

Prevent future waste.

Examples:

- Idle resource detection.
- Scheduling non-production workloads.
- Automated cleanup.

Senior perspective:

Cost optimization is an engineering responsibility, not only a finance activity.

---

## Q69. How do you balance reliability and cost?

Answer:

I avoid optimizing one dimension.

Example:

Cheapest architecture:

May create reliability risks.

Most redundant architecture:

May create unnecessary cost.

Decision process:

Understand:

- Business requirements.
- Availability targets.
- Recovery objectives.

Then choose:

- Appropriate redundancy.
- Appropriate capacity.
- Appropriate operational effort.

---

## Q70. What does good AWS platform engineering look like?

Answer:

A good AWS platform provides:

Developer experience:

- Self-service infrastructure.
- Clear golden paths.
- Fast delivery.

Reliability:

- Automation.
- Observability.
- Recovery processes.

Security:

- Secure defaults.
- Governance.
- Identity management.

Efficiency:

- Cost visibility.
- Standardization.

Success metrics:

- Deployment speed.
- Reliability.
- Developer adoption.
- Reduced operational toil.

The platform team's goal is enabling engineers to ship safely and quickly.

---
---

## Q71. How would you design AWS infrastructure for an internal developer platform?

Answer:

I would treat AWS infrastructure as a platform capability rather than individual cloud resources.

The design should provide:

Self-service:

- Developers consume approved infrastructure patterns.
- Infrastructure is provisioned through templates/modules.
- Avoid manual AWS console operations.

Standardization:

- Terraform modules.
- Approved networking patterns.
- Standard IAM roles.
- Required tagging.

Security:

- Least privilege IAM.
- Centralized logging.
- Encryption by default.
- Security guardrails.

Operations:

- Monitoring.
- Alerting.
- Automated remediation.
- Disaster recovery processes.

Developer workflow:

Developer request

->

Platform template

->

Terraform automation

->

Security validation

->

AWS resource provisioning

->

Observability enabled

The goal is reducing developer cognitive load while maintaining enterprise controls.

---

## Q72. How do you manage AWS infrastructure across multiple accounts?

Answer:

At enterprise scale, multiple AWS accounts are required for isolation and governance.

Typical structure:

- Production accounts.
- Development accounts.
- Security account.
- Logging account.
- Shared services account.

Controls:

Identity:

- Central SSO.
- IAM roles.
- Permission boundaries.

Governance:

- AWS Organizations.
- Service Control Policies.
- Account vending automation.

Infrastructure:

- Standard Terraform modules.
- Version-controlled infrastructure.
- Automated compliance checks.

Operational model:

Developers should not request infrastructure through tickets.

They should consume approved platform capabilities.

---

## Q73. How do you create self-service AWS infrastructure for developers?

Answer:

The goal is to expose capabilities, not raw cloud resources.

Poor approach:

"Here is AWS documentation. Create your own infrastructure."

Better approach:

Provide:

- Terraform modules.
- Templates.
- Internal APIs.
- Developer portals.
- Automated workflows.

Example:

Developer selects:

"Create application environment"

Platform automatically creates:

- IAM role.
- Networking.
- Compute resources.
- CI/CD integration.
- Monitoring.
- Security controls.

Benefits:

- Faster delivery.
- Consistent architecture.
- Reduced support tickets.
- Better compliance.

---

## Q74. How do you implement AWS security without slowing developers down?

Answer:

Security should be built into the platform.

Instead of:

Developer creates resource

then

Security reviews later

Use:

Secure defaults.

Examples:

Infrastructure:

- Approved Terraform modules.
- Encrypted storage.
- Private networking.

Identity:

- Short-lived credentials.
- IAM roles.
- Least privilege access.

CI/CD:

- Security scanning.
- Policy validation.
- Dependency checks.

The best security controls are invisible to developers while preventing unsafe patterns.

---

## Q75. How do you control AWS cloud costs in a platform environment?

Answer:

Cost optimization should be part of platform engineering.

Visibility:

- Resource ownership tags.
- Cost dashboards.
- Application-level attribution.

Optimization:

- Right-size workloads.
- Remove unused resources.
- Automate cleanup.
- Use appropriate purchasing models.

Platform controls:

- Standard instance choices.
- Resource quotas.
- Budget alerts.
- Cost policies.

Measurement:

- Cost per application.
- Cost per environment.
- Infrastructure utilization.

The goal is not simply reducing spending.

The goal is improving engineering efficiency per dollar.

---

## Q76. How do you manage IAM at scale in AWS?

Answer:

IAM should be designed around roles and automation.

Best practices:

Avoid:

- Shared users.
- Long-lived credentials.
- Excessive administrator access.

Use:

- Identity federation.
- IAM roles.
- Temporary credentials.
- Permission boundaries.

For applications:

Application

->

IAM Role

->

AWS permissions

not:

Application

->

Hardcoded access keys

Governance:

- Access reviews.
- CloudTrail auditing.
- Automated policy checks.

---

## Q77. How would you design AWS networking for a developer platform?

Answer:

The platform should provide secure network patterns by default.

Architecture:

Public layer:

- Load balancers.
- External entry points.

Private layer:

- Applications.
- Services.
- Workers.

Restricted layer:

- Databases.
- Sensitive systems.

Best practices:

- Multi-AZ design.
- Controlled routing.
- Security groups.
- Network monitoring.
- VPC endpoints where appropriate.

Developers should consume approved network patterns instead of designing networking from scratch.

---

## Q78. How do you troubleshoot AWS production issues?

Answer:

I start from customer impact and work inward.

Step 1:

Identify impact:

- Which service?
- Which users?
- What changed?

Step 2:

Check platform signals:

- Metrics.
- Logs.
- Traces.
- Alerts.

Step 3:

Validate AWS components:

- Compute.
- Networking.
- IAM.
- Storage.
- Managed services.

Step 4:

Mitigate:

- Roll back.
- Scale.
- Fail over.
- Disable problematic changes.

Step 5:

Prevent recurrence:

- Automation.
- Monitoring improvements.
- Documentation.
- Architecture changes.

Senior engineers focus on reducing MTTR through better systems.

---

## Q79. How do you design disaster recovery for AWS workloads?

Answer:

Disaster recovery depends on business requirements.

First define:

RTO:

How quickly must service recover?

RPO:

How much data loss is acceptable?

Strategies:

Backup and restore:

- Lower cost.
- Longer recovery.

Warm standby:

- Partial environment running.

Multi-region:

- Active recovery capability.

Implementation:

- Automated backups.
- Infrastructure as Code.
- Tested recovery procedures.
- Documented runbooks.

A backup strategy is incomplete until recovery has been tested.

---

## Q80. What AWS platform engineering principles do you follow?

Answer:

My AWS platform principles are:

1. Self-service over ticket-driven operations.

Developers should safely consume infrastructure without waiting on platform teams.

2. Secure defaults.

Security should be built into templates and workflows.

3. Infrastructure as Code.

All infrastructure should be version-controlled and reviewable.

4. Automation first.

Remove repetitive operational work.

5. Golden paths.

Provide recommended ways to build and deploy.

6. Observability by default.

Every service should have monitoring from day one.

7. Measure outcomes.

Track:

- Deployment frequency.
- Lead time.
- Change failure rate.
- MTTR.
- Developer productivity.

A successful AWS platform enables teams to deliver software faster while improving reliability, security, and operational efficiency.

---
