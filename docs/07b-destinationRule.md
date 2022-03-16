# Destination rule

Its again another Istio resource.

Destination rule creates the destination group based on the labels and provides a name, which is then referred by virtualService.

Definition: Destination rule are configuration for *Istio load balancer*. For canary we create subsets, which pod should be part of each subset.

```yaml
kind: DestinationRule
apiVersion: netwroking.istio.io/v1alpha3
metadata:
  name: somename
spec:
  host: fleetman-staff-service.default.svc.cluster.local # after reaching the service, use the below pod selector
  subsets:
    - labels: # pod selector
        version: safe
      name: safe
    - labels: # pod selector
        version: risky
      name: risky
```

## Traffic policy and consistent hashing

Consistent hashing is load balancing algorithm, where some data(like cookie or header or anything else) are hashed using a hash algorithm and then based on the hashed value we perform the load balancing.

```yaml
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
  subsets:
    - labels: # pod selector
        version: safe
      name: safe
    - labels: # pod selector
        version: risky
      name: risky
```

> VERY IMPORTANT: The above will NOT work, no stickyness(session affinity) is generated because of the source IP address above. Explained in issue 9764 in istio. Based on the issue, 9764, session affinity(using traffic policy consistentHash) do NOT work with weighted subsets because this is not supported by Envoy. Now my explantion below:


                          >>>>>>>>   LOAD       >>>>>>>>> POD replica(s)(canary)
                                    BALANCER

CLINT >>>>>  WEIGHTING
              SUBSET


                          >>>>>>>>   LOAD       >>>>>>>> POD replica(s)(non-canary)
                                    BALANCER

> As you can see above, envoy, first applies the weighting subset(and the traffic is randomly splitted based on the weight) and then it applies the load balancing rules. hence as you can understand stickyness will not work.

THEN WHY TO USE CONSISTENT HASHING? IF IT DO NOT WORK AS EXPECTED

Consistent hash do work for the below(VVVV) diagram when we DO NOT have weighting subsets.

                        >>>>>>>>> POD replica(s)(canary)

CLINT >>>>>  LOAD
            BALANCER

                        >>>>>>>> POD replica(s)(non-canary)

> NOT load balancer will apply the hash and split the traffic based on the hash. It do not matter what you use for hasing, it can be any of the defined theing in Istio(cookie, ip or header)

## Using header as hashing

> ANOTHER IMPORANT TRICKS: This will not work because there are multiple microservice(including the proxy) and header need to be propagated properly through the chain of microservices.

As well NOTE: each user should have different value of the header
