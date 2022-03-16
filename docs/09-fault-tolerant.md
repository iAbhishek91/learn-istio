# Injecting fault tolerent

Netflix's idea of chaos engineering of introducing fault of the system and analyse how system behave.

Below yaml will suspend traffic to any underlying service.

```yaml
kind: VirtualService
apiVersion: networking.istio.io/v1alpha3
metadata:
  name: fleetman-vehicle-telemetry
  namespace: default
spec:
  hosts:
    - fleetman-vehicle-telemetry
  http:
    - fault:
        abort: # we can also provide "delay" or "abort"
          httpStatus: 503
          percentage:
            # 100 will block all the traffic to pass(creating a fault in the system)
            # less than 100 will create an inconsistent service
            value: 100
      route:
        - destination:
            host: fleetman-vehicle-telemetry
```

>NOTE: there is no subset defined, and so no destinationRule requried. Sometime people create a destinationRule along with virtualService, in that case we can create a destination rule without any subset.

We can also provide **match** to perform some match with the header or cookies.

Below is an example of injecting fault of latency

```yaml
kind: VirtualService
apiVersion: networking.istio.io/v1alpha3
metadata:
  name: fleetman-staff-service
  namespace: default
spec:
  hosts:
    - fleetman-staff-service
  http:
    - match:
        - headers:
            x-my-header:
              exact: canary
      fault:
        abort:
          percentage:
            value: 100.0
        httpStatus: 418 # BE CAREFUL WITH INDENTATION!
      route:
        - destination:
            host: fleetman-staff-service
            subset: risky
    - route:
        - destination:
            host: fleetman-staff-service
            subset: safe
---
kind: DestinationRule
apiVersion: networking.istio.io/v1alpha3
metadata:
  name: fleetman-staff-service
  namespace: default
spec:
  host: fleetman-staff-service
  subsets:
    - labels:
        version: safe
      name: safe
    - labels:
        version: risky
      name: risky
```
