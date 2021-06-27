# Gateway

As per Istio, it suggest to replace the "ingress resource" with "ingress gateway".

WHY?

## Why virtualService with weighted segment do NOT work for first service

First service is the servie which is exposed as a "Nodeport" or as exposed to ingress gateway.

And that do NOT work with weighted segment (implemented within the VirtualService and Destination rules), unlike other internal "clusterIP" service where it works.

This is because, "Nodeport" services or ingress are directly hit by client(browser or curl  ...), hence it do not pass through any proxy, proxies are intercepted only when a pod communicate with other pod managed by istio.

As per docs the solution to the above problem is: "Edge Proxy" from envoy, whenever we are coming in a cluster managed by istio.

> Trick is Istio Ingress GATEWAY implement an edge proxy, which is missing in normal Ingress resource provided by Nginx. This is NOT mentioned in the Istio docs, however Envoy docs makes it very very clear.
> As well I reckon, GATEWAY can be mappd with VirtualService, but notmal service would not.

## How do we implement ISTIO INGRESS GATEWAY

Istiod comes along with **ingressgateway**(name: istio-ingressgateway-6cfd75fc57-8jrwj), which is found in istio-system namespace. Also a service to expose the **ingress gateway**(name: istio-ingressgateway) of type LoadBalancer with several ports(15021:32000/TCP,80:31380/TCP,443:32002/TCP,31400:32003/TCP,15443:32004/TCP)

This service points to the default istio ingress pod.

Its an easy YAMl that we need to deploy along side the cluster.

The port is **31380** by default and we can use that once our Gateway is deployed.

There is generally one gateway implemented for the entire system.

```yaml
apiVersion: nertowkring.istio.io/v1alpha3
kind: Gateway # this configures the istio ingress configuration
metadata:
  name: ingress-gateway-configuration
spec:
  selector:
    # this is one of the label in ingressGateway pod, for which we are configuring the gateway
    # which ingress gateway container are we configuring, we already have the ingress gateway pod, this is done as there may be multiple ingress-gateway pod, check for the containers in that
    istio: ingressgateway 
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*" # proper gateway name, else leave it open for all (*)
```

### Understanding the data flow

Refer: 07c-traffic-flow-via-gateway.png

- data comes to the loadbalancer/service URL *in minikube the loadbalancer is not configured, but we can use one of the port in the service(istio-ingressgateway)*
- the service/LB points to the istio-ingressgateway pod.
- Istio ingress container in the pod forward the request to "Edge proxy" *this is implemented by Istio using Gateway*
- The Edge proxy has VirtualService and Destination rule configured, which forwards the trafic to the configured destination. *NOTE: the virtual service has link to gateway that is used*
- The application pod receives the traffic.
