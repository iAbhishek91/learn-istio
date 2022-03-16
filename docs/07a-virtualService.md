# VirtualService

Its a Istio resource, which is used for weighted routing(canary).

Another definition: a VirtualService enables us to configure routing rules to the service mesh.

Much like ingress, VirtualService has destination, with subset and weight for weighted routing.

Virtual service works in conjunction with destination rules. Destination rule creates the destination group based on the labels and provides a name, which is then referred by virtualService.

```yaml
kind: VirtualService
apiVersion: networking.istio.io/v1alpha3
metadata:
  name: somename # for consistency keep it same as the service
spec:
  hosts:
    - fleetman-staff-service.default.svc.cluster.local # from where requests are coming, this service will forward to the app. To be explicit we're applying regular kubernetes service name. that we are applying the routing rules to. It may be something coming from externally.
  http:
    - route:
        - destination:
            host: fleetman-staff-service.default.svc.cluster.local # where the requests are going to
            subset: safe # this name should match from Destination rules
          weight: 0 # should add upto 100 for single route
        - destination:
            host: fleetman-staff-service.default.svc.cluster.local
            subset: risky # this name should match from Destination rules
          weight: 100 # sould add upto 100 for single route
```

FOR VISUALISATION, **WHERE DOES A VIRTUALSERVICE FITS IN KUBERNETES**

- When we create a virtual service, we send these to IstioD or Pilot in control-plane, and in turn Istiod will distribute those configuration to proxies.
- Under the hood, the job is done by Envoy, Istio is just distributing those configuration to all the Envoy proxy in an efficient manner.
- Kubernetes services(service discovery) are no where replaced by Istio VirtualService(as it applies routing rules using label selectors). The envoy proxy still need to use kubernetes service to identify the pods however it can apply some dynamic rules.
- OPTIONAL: only if you want to configure dynamic routing rules, then only virtual service is required.

> NOTE: K8S service are nothing to do with Istio VirtualService. They are completely independent, different and unrelated.

For Consistent Hasing the below YAML

```yaml
kind: VirtualService
apiVersion: networking.istio.io/v1alpha3
metadata:
  name: somename # for consistency keep it same as the service
spec:
  hosts:
    - fleetman-staff-service.default.svc.cluster.local # from where requests are coming, this service will forward to the app
  http:
    - route:
        - destination: # single destination for stickiness
            host: fleetman-staff-service.default.svc.cluster.local # where the requests are going to
            subset: all-pods # this name should match from Destination rules
          weight: 100 # optional for single destination
---
kind: DestinationRule
apiVersion: netwroking.istio.io/v1alpha3
metadata:
  name: somename
spec:
  host: fleetman-staff-service.default.svc.cluster.local # after reaching the service, use the below pod selector
  trafficPolicy: # define the policy of the load balancer how traffic will be forwarded
    loadBalancer:
      consistentHash:
        useSourceIp: true # here we are using source IP for hashing
        #httpHeaderName: "myHeader" # another option of using header,  in this case header propagation is very very important
  subsets:
    - labels: # only one pod selector
        app: staff-selector
      name: all-pods
```

While using virtual service along with Gateway, use the below YAML:

```yaml
kind: VirtualService
apiVersion: networking.istio.io/v1alpha3
metadata:
  name: fleetman-webapp
  namespace: default
spec:
  hosts:
  # which incoming host are we applying the proxy rules to???
  # Copy the value in the gateway hosts - usually a Domain Name
  # Basically, we need to mention all the hosts that can communicate with the targeted pod
  # this icludes all the service that internally communicate with the pod, as well as the external domain name(thats the reason we copy all the hosts name from the Gateway)
    - "*" 
  gateways:
    - ingress-gateway-configuration # NAME of the gateway, THIS IS IMPORTANT here
  http:
    - route:
        - destination:
            host: fleetman-webapp
            subset: original
          weight: 90
        - destination:
            host: fleetman-webapp
            subset: experimental
          weight: 10
```

While using virtual service for path based routing

We can matches based on route, headers etc

> NO change requried in Destination Rules and Gateway

```yaml
kind: VirtualService
apiVersion: networking.istio.io/v1alpha3
metadata:
  name: fleetman-webapp
  namespace: default
spec:
  hosts:      # which incoming host are we applying the proxy rules to???
    - "*"
  gateways:
    - ingress-gateway-configuration
  http:
    - match:
      - uri:  # IF
          prefix: "/experimental"
      - uri:  # OR
          prefix: "/canary"
      route: # THEN
      - destination: # we can do weighted subset as well with prefix
          host: fleetman-webapp
          subset: experimental
    - match:
      - uri :
          prefix: "/"
      route:
      - destination:
          host: fleetman-webapp
          subset: original
---
#domain name based matching and routing
#here we configure multiple virtualService for each host(which is easy & simple to configure)
# For local learning purpose make use of local hosts file to create DNS
kind: VirtualService
apiVersion: networking.istio.io/v1alpha3
metadata:
  name: fleetman-webapp-experiment
  namespace: default
spec:
  hosts:      # which incoming host are we applying the proxy rules to???
    - "experimental.fleetman.com"
  gateways:
    - ingress-gateway-configuration
  http:
      - route:
        - destination:
            host: fleetman-webapp
            subset: experimental
```

HEADER based routing are explained under xx-dark-release

Apart from all the conditional routing we can mention a catch all condition,  we can also provide name(useful for documentation)

```yaml
kind: VirtualService
apiVersion: networking.istio.io/v1alpha3
metadata:
  name: fleetman-webapp
  namespace: default
spec:
  hosts:      # which incoming host are we applying the proxy rules to???
    - "*"
  gateways:
    - ingress-gateway-configuration
  http:
    - name: canary # optioanl name to the routing condition
      match:
      - headers:  # IF
          my-header:
            exact: canary
      route: # THEN
      - destination:
          host: fleetman-webapp
          subset: experimental
    - name: safe # optional name to the routing condition
      route: # CATCH ALL(which do not match any)
      - destination:
          host: fleetman-webapp
          subset: original
```
