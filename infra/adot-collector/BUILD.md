# ADOT Collector – Build & Push (CloudShell)

This documents how to manually build and push the ADOT collector image from **AWS CloudShell**.

Use this when:
- you update `otel-config.yaml`
- you need to fix scrape/export behavior
- you want to cut a new collector image without touching the app pipeline

---

## Prerequisites

- You are logged into **AWS CloudShell**
- You are in the repo root
- ADOT files live at:
  ```
  ./infra/adot-collector/
  ├── Dockerfile
  └── otel-config.yaml
  ```
- AWS region: `us-west-2`
- ECR repo: `jay-platform/adot-collector`

---

## 1. Set environment variables

```
AWS_REGION=us-west-2
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
REPO_NAME=jay-platform/adot-collector
IMAGE_TAG=git-initial
```

---

## 2. Ensure the ECR repository exists

```
aws ecr describe-repositories   --repository-names $REPO_NAME   --region $AWS_REGION   >/dev/null 2>&1 || aws ecr create-repository   --repository-name $REPO_NAME   --region $AWS_REGION
```

---

## 3. Login Docker to ECR

```
aws ecr get-login-password --region $AWS_REGION | docker login   --username AWS   --password-stdin ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
```

Expected output:
```
Login Succeeded
```

---

## 4. Build the ADOT collector image

```
cd infra/adot-collector

docker build   -t adot-collector:${IMAGE_TAG}   .
```

---

## 5. Tag the image for ECR

```
docker tag adot-collector:${IMAGE_TAG}   ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${REPO_NAME}:${IMAGE_TAG}
```

---

## 6. Push the image to ECR

```
docker push   ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${REPO_NAME}:${IMAGE_TAG}
```

---

## After Push

1. Update the ECS task definition to reference the new image tag:
   ```
   <account>.dkr.ecr.us-west-2.amazonaws.com/jay-platform/adot-collector:<IMAGE_TAG>
   ```
2. Redeploy the service (via CodePipeline / CodeDeploy).
3. Verify ADOT logs:
   - Prometheus scrape succeeds
   - No 403s on AMP remote write
   - Metrics appear in AMP / Grafana

---

## Notes

- This image bakes in `otel-config.yaml`. Any config change requires a rebuild.
- Consider pinning the base image tag instead of `:latest` for reproducibility.
- Long term improvement: move AMP endpoint to SSM Parameter Store.
