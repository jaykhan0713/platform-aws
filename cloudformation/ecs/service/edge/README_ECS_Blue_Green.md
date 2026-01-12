## ECS Blue/Green Deployment with CodeDeploy and CodePipeline

This service uses **ECS Blue/Green deployments powered by AWS CodeDeploy**, with deployments orchestrated through **AWS CodePipeline**. The setup enables zero-downtime releases, controlled traffic cutover, and safe rollback windows.

---

### Architecture Overview

- **ECS Fargate service** configured with `DeploymentController = CODE_DEPLOY`
- **Internal Application Load Balancer (ALB)** with:
  - Production listener on port `80`
  - Test listener on port `9000`
  - Blue and Green target groups
- **AWS CodeDeploy** application and deployment group managing task sets and traffic shifting
- **AWS CodePipeline** handling build artifacts and initiating deployments

---

### End-to-End Deployment Flow

#### 1. Build Stage

- CodeBuild builds the Docker image and pushes it to Amazon ECR.
- The build produces two deployment templates:
  - `taskdef.json` – ECS task definition template
  - `appspec.yaml` – CodeDeploy AppSpec file

---

#### 2. Task Definition Templating

- `taskdef.json` is generated from the currently running ECS service task definition.
- Describe-only fields (for example `revision`, `status`, `taskDefinitionArn`) are stripped so the task definition is registerable.
- The container image is updated to the newly built ECR image URI.
- During deployment, **CodePipeline registers a new ECS task definition revision**.

---

#### 3. CodeDeploy AppSpec

The AppSpec file uses the required placeholder:

```yaml
version: 0.0
Resources:
  - TargetService:
      Type: AWS::ECS::Service
      Properties:
        TaskDefinition: <TASK_DEFINITION>
        LoadBalancerInfo:
          ContainerName: app
          ContainerPort: 8080
```

- `<TASK_DEFINITION>` is replaced by CodeDeploy with the newly registered task definition ARN.
- CodeDeploy creates a **new ECS task set** attached to the Green target group.

---

#### 4. Traffic Shifting and Validation

- The Green task set is brought up behind the **test listener**.
- ALB health checks are performed on the new tasks.
- Once healthy, production traffic is shifted to the Green target group.
- This service uses `CodeDeployDefault.ECSAllAtOnce`, meaning traffic is switched immediately after health checks pass.

---

#### 5. Blue Task Set Termination

- After traffic cutover, the old Blue task set remains running for a configurable safety window.
- This repository uses a **5-minute termination wait time**, allowing quick rollback if needed.
- After the wait period, CodeDeploy automatically terminates the old task set.

---

### Why CodePipeline Registers Task Definitions

With ECS Blue/Green deployments, **task definitions must be registered explicitly** for each deployment. Unlike rolling ECS updates, CodeDeploy operates on task sets and requires a new task definition ARN per release.

Because of this:
- The CodePipeline deploy role must be allowed to call `ecs:RegisterTaskDefinition`.
- CodeDeploy then uses the registered ARN to orchestrate the deployment lifecycle.

---

### Key Takeaways

- ECR stores container images, ECS registers task definitions that reference those images, and CodeDeploy consumes the resulting task definition ARN to orchestrate blue/green task sets.
- Blue/Green deployments create **separate task sets**, not in-place task replacements.
- `<TASK_DEFINITION>` is handled by CodeDeploy and represents the new task definition ARN.
- CodePipeline is responsible for registering task definitions and initiating deployments.
- Old task sets are retained temporarily for rollback safety, then cleaned up automatically.

This setup provides deterministic, production-grade deployments with zero downtime and clear rollback semantics.
