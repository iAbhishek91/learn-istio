# Telemetry

Its about gathering metrics and populating that to gain more meaninful insights.

## Minimum to requirement to have telemetry

- Envoy proxy sidecar container injected to the target pods.
- Istio control-plane needs to be running - istiod, kiali, jaeger, grafana
- NO NEED of any specific istio yaml configuration: Virtual Services and Gateway
