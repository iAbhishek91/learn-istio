# What is Istio

- Istio is service mesh

## What is service mesh

- its is a layer of software you deploy alongside your K8s cluster(it can be any cluster where microservice based architecture is deployed.)
- All the networking calls are routed through the layer(service mesh), so that it can take charge of networking. (Pods instead of invoking another pod(service) it sends the request to service mesh, service mesh then forwards the request to the target pod)
- In between service mesh may have a "Mesh logic" to be applied on the request or on the response.

## Why Istio is requried

- Kubernetes is good at scheduling and managing pods, however, it is Not so good at depecting the relation between the Pods.
- Also its not feasible to infer whats going on inside the network calls b/w the pods.

## How Istio implements a service mesh

- There are thousand different way to implement a mesh, Istio **injects** a proxy container (inside the pod as a sidecar)
  - The proxy pod is NOT implemented by Istio. It comes from a open source project called [envoy](https://envoyproxy.io), created by "Lyft"(similar to Uber). *Envoy is not specific to k8s, it can run on many different cluster tools*.
  - The main container sends all the network call to the Istio proxy container.
  - The proxy container contains the "Mesh Logic".
  - The proxy container under the hood uses IP tables magic to do all the above.
  - All the proxy contianer, collectively known as *Istio Data Plane*.
- Prior to 1.15, there were lots of pods created in namespace **istio-system**, with different responsibility. However, later the Istio was refactored and **Istiod** pod was created(previously known as *pilot*). This pod runs most of Istio.
- The Proxy pod, forwards request to target pods, simultaneously forward request to IstioD to perform other job(telemetry for example).
- There are few other pods running on **istio-system** namespace. currently in 1.18 version there are 7 pods to be specific
  - *istio-egressgateway* : control traffic going out from the pod
  - *istio-ingressgateway*: control traffic coming in to the pod
  - *istio-tracing(renamed to jaeger)*: used for tracing.
  - *istiod*: major part of Istio(consolidated now)
  - *kiali*: visualization UI for Istio
  - *prometheus*: monitoring
  - *grafana*: monitoring dashboard

## What changes are requried to make on the containers to implement Istio

Generally we dont need to do any changes on the application code. Istio or service mesh in general is something that we switch on or switch off without any overhead.

## How to inject proxy-containers in the application pods

Your application should run a sidecar of Istio-proxy contianer. But how do we inject that in the YAML.

Well there are many differnt ways to achieve that,

- we can add a container to all the pod manifest(not recommended as it will alter the pod yamls)
- Allow Istio to inject proxy pod to all the pods on a perticular namespace. *this is done by istio-sidecar-injector in previous version, now post 1.15 is done by istiod pod*
  - this is done by injecting  a label **istio-injection=enabled** in all the the namespace, we want Istio to work. *k label ns default istio-injection=enabled*
  - all pod will go through some setup *this is done by init contianer*, to verify you can see a additional container for each pod under the namespace.

>NOTE: if you forgot to add the label, or namespace and application is already running, in that case we have to somehow restart the pod after labeling.
