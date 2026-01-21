FROM public.ecr.aws/aws-observability/aws-otel-collector:v0.46.0

ENTRYPOINT ["/awscollector"]
CMD ["--config", "env:OTEL_CONFIG_YAML"]
