# Blackhole and Passthrough cluster

This are concept associated with external and internal service.

**External service** are services that are not part of the service mesh, and **Internal services** are those which are part of the service mesh.

For **external service there is two options**:

- Block all external service: outboundTrafficPolicy.mode = REGISTRY_ONLY
- Allow all internal service: outboundTrafficPolicy.mode = ALLOW_ANY

**Blackhole** and **Passthrough** cluster are virtual cluster created in the Envoy configuration **Based on the Istio sidecar configuration**

For **REGISTRY_ONLY** blackhole cluster is created.
and For **ALLOW_ONLY** passthrough cluster is created.

## Blackhole cluster

