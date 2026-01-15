# platform-aws
AWS platform architecture consisting of AWS templates, diagrams, workflows, and backlogs.

---
## VPC
VPC baseline for jay-platform (prod)
- 2 AZs (us-west-2a / us-west-2b)
- Private subnets for edge-service + internal ALB, ECS workloads
- VPC Endpoints, no NAT
- TODO: Add more descriptions
---

## Workflow

```
Client Request

-> Http API Gateway w/ Cognito Authorization attached to integrations

-> VPC Link 

-> internal ALB on private subnet

-> edge-service (ECS Fargate)

-> internal services via Service Connect

```
--- 

## Source/Build/Deploy time

- edge-service is deployed via CodePipeline, uses blue-green CodeDeploy Controller.
- CodePipeline itself and Source + Build is not IaC yet for edge-service but planning to for internal services.
- CodeDeploy BG configs (tg, controller) related are IaC under edge-service. Actual node in pipeline is not IaC yet. 

---

## Observability (WORKING END TO END)

This was a big one to wire esp with CodeDeploy.

adot-collector: TODO

---

## ECR extras

- I've pushed stable images for temurin jdk and jre (25) in ECR as docker has download limits.
- Sidecar adot collector is a manual image as well as it's a stable image that does what I need it to do at this point.

---

## IaC decisions

Note that mostly everything living inside and related to VPC is available as 
cloudformation IaC templates in this repo to keep portability as much as possible. This gives me
some piece of mind for when I want to rebuild my infra in a day say, to have multi-account based (test+prod) + 
platform account for codepipeline but that would cost a lot of $$ for personal projects :')
but maybe one day! :)

