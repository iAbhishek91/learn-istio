# Dark Releases

This is NOT a feature of Istio, we refer to Dark release as a release which can be deployed without testing. And this can be achieved via Istio.

We have already seen:

- PATH based routing
- DOMAIN based routing

HEADER based routing is something we are going to implement to achieve Dark Release.

## HEADER based routing

Header based matching can be done on below condition

- exact: "value"
- prefix: "value"
- regex: "value"

> NOTE: Headers are case-sensetive

After deploying till example/_course_files/4 Dark Releases/ 6

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
      - headers:  # IF
          my-header:
            exact: canary
      route: # THEN
      - destination:
          host: fleetman-webapp
          subset: experimental
    - route: # CATCH ALL
      - destination:
          host: fleetman-webapp
          subset: original
```

try below for CURL, from browser use **modheader**

```sh
# FOR catch all scenario
curl 192.168.49.2:31380

# FOR CANARY Release
curl 192.168.49.2:31380 --header "my-header:canary"
```

----------------------------------------------------------------

Till here we were able to achieve Dark Release of initial app via HEADER matching. How do we do Dark Release of all the microservices which are internal? - simple, header propagation is important via application and apply the change in the Virtual service.
