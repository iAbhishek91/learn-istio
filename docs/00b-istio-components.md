# Istio Components

Istio is logically divided into control plane and data plane.

## control Plane

**IstioOperator** a k8s operator which deploys all the control plane components, along with provided configuration. IstioOperator is explained in `00c-istioOperator.md`. 

*Based on the configuration/profile(built-in) configuration optional components will be installed*.

1. **Istio core** [required]

2. **IstioD** [required]: Combination of all below components
  
  a. **Pilot**: configs data to Envoys
  b. **Mixer**: Policy and telemetry
  c. **Citadel**: TLS certs to Envoys
  d. **Sidecar Injector**: Monitor K8s for new pods to inject Envoys

3. **Ingress Gateway** [optioanl]

4. **Egress Gateway** [optional]

## data plane

1. **Envoy/sidecar/proxy**: in each pods in the mesh

    a. **istio-agent**: this is part of the sidecar really, but is a separate component.
