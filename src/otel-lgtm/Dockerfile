FROM grafana/otel-lgtm:latest

COPY ./config/otelcol-config-novolume.yaml /otel-lgtm/otelcol-config.yaml
COPY ./config/tempo-config.yaml /otel-lgtm/tempo-config.yaml
COPY ./config/grafana-dashboards.yaml /otel-lgtm/grafana/conf/provisioning/dashboards/grafana-dashboards.yaml
COPY ./dashboards /otel-lgtm/dashboards
