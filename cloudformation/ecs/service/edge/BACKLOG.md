- Health liveness + readiness check
- Remove taskdefinition handling and keep it to pipeline since 
  in ALB CodeDeploy controller path: CodeDeploy is in charge of creating taskdef revisions via
  service's defined taskdef.json