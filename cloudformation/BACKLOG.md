## Cloudformation Backlog

- change outputs + parameters to exporting once architecture is fully decided and stable.
- Template Project/Env/ManagedBy tags so no hardcoding everywhere.

| Order | Area               | Goal                                         | What you create                                                                              | Where it lives              | “Done” check                                                   |
| ----: | ------------------ | -------------------------------------------- | -------------------------------------------------------------------------------------------- | --------------------------- | -------------------------------------------------------------- |
|     0 | Auth               | Reset platform auth cleanly                  | Cognito User Pool, App Client, (optional) Domain, Resource Server + scopes, User Pool Groups | Cognito                     | You can get a valid JWT for yourself and see claims            |
|     1 | Edge routing       | Enforce auth at the edge                     | HTTP API JWT authorizer + attach to routes (`ANY /` and `ANY /{proxy+}`)                     | API Gateway HTTP API        | Unauthed request returns 401, authed request hits edge-service |
|     2 | Network hardening  | Tighten ingress to ALB                       | Change ALB SG ingress from VPC CIDR to **VPC Link SG** on port 80                            | Edge-service stack (ALB SG) | API Gateway still works, random VPC traffic no longer allowed  |
|     3 | Observability UI   | Get Grafana UI ready                         | Amazon Managed Grafana workspace                                                             | AMG                         | You can log into Grafana workspace                             |
|     4 | Metrics backend    | Create metrics sink                          | Amazon Managed Prometheus workspace                                                          | AMP                         | Workspace exists, you have the remote write URL                |
|     5 | Grafana datasource | Connect Grafana to metrics                   | Add AMP datasource in Grafana                                                                | AMG                         | “Save and test” succeeds                                       |
|     6 | Metrics shipping   | Actually get `/actuator/prometheus` into AMP | ADOT collector (sidecar per service preferred) scraping actuator and remote-writing to AMP   | ECS task definition         | You see your service metrics in Grafana (JVM + http metrics)   |
|     7 | Tracing backend    | Get traces into AWS                          | ADOT collector exports OTLP to **X-Ray** (recommended)                                       | ECS task definition + X-Ray | Traces show up in X-Ray service map / traces list              |
|     8 | Dashboards         | Make it look portfolio-ready                 | A couple Grafana dashboards + alarms (optional)                                              | AMG + CloudWatch            | You have a dashboard link you can share                        |

done: 1, 3, 4, 5, 6, 7, 
todo: 2
in-progress: 8- has golden signals so far, needs more work.