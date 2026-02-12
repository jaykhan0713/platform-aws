
##Migration

---
task role:

AllowApsRemoteWrite:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "aps:RemoteWrite"
            ],
            "Resource": "arn:aws:aps:us-west-2:454842419646:workspace/ws-2eea101e-6aad-4579-a198-d29198b23426",
            "Effect": "Allow",
            "Sid": "ApsRemoteWrite"
        }
    ]
}
```

AllowEcsExecSsmMessages:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "ssmmessages:CreateControlChannel",
                "ssmmessages:CreateDataChannel",
                "ssmmessages:OpenControlChannel",
                "ssmmessages:OpenDataChannel"
            ],
            "Resource": "*",
            "Effect": "Allow",
            "Sid": "EcsExecSsmMessages"
        }
    ]
}
```

AllowSsmParametersRead:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "ssm:GetParameter",
                "ssm:GetParameters",
                "ssm:GetParametersByPath"
            ],
            "Resource": [
                "arn:aws:ssm:us-west-2:454842419646:parameter/jay-platform/prod/core/*",
                "arn:aws:ssm:us-west-2:454842419646:parameter/jay-platform/prod/services/edge-service/*"
            ],
            "Effect": "Allow",
            "Sid": "ReadParamsByPath"
        }
    ]
}
```

AWSXrayWriteOnlyAccess:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "xray:PutTraceSegments",
                "xray:PutTelemetryRecords",
                "xray:GetSamplingRules",
                "xray:GetSamplingTargets",
                "xray:GetSamplingStatisticSummaries"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```