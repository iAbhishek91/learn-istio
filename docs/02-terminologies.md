# Terms associated with Istio

- Control Plane: Collectively whats running on istio-system namespace is referred as Control Plane.
- Data Plane: The proxy pods are collectively called Data plane in Istio.
- Jaeger: UI used for tracing pod, located control plane.
- Kiali: UI for having a overview of Istio
- Telemetry: is basically gathering of mertices and representing things on UI, it includes Kiali, Jaeger and Grafana.
- Trace: tracing entire request and response is known as trace
- Span: each section in a trace is called span
- Upstream pods: when a pod request another pod (that request is called upstream request)
- downstream pods: when a pod response to another pod(that request is called downstream request)
