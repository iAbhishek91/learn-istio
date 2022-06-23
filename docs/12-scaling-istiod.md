# Factors affecting scaling of IstioD

There are few factors to be considered

## Size of config to generate

config includes - listeners, clusters and routes. bigger the cluster bigger the config, hence its important to narrow down the scope using SideCars

## Rate of change of environment

- Every time a new services is created or istio configuration is changed full updates are sent to proxy/sidecars
- Adding new endpoints are cheap as incremental updates are sent to Istio

## Number of proxy impacted by a change

- change in configuration(vs, dr, sidecar) and services are pushed to all the pods
- keep configs bound to namespaces, instead of global if they are changed frequently
- keep number of pods/services as few possible.
