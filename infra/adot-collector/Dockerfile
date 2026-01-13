FROM public.ecr.aws/aws-observability/aws-otel-collector:latest

COPY otel-config.yaml /etc/otel-config.yaml

ENTRYPOINT ["/awscollector"]
CMD ["--config=/etc/otel-config.yaml"]
