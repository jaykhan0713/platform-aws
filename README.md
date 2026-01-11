# platform-aws
AWS platform architecture consisting of AWS templates, diagrams, workflows, and backlogs.

---
## VPC
VPC baseline for jay-platform (prod)
- 2 AZs (us-west-2a / us-west-2b)
- Private subnets for edge-service + internal ALB, ECS workloads
- VPC Endpoints, no NAT
---
