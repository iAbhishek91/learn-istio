# Debugging Istio

As we know that Istio is separated in **Data plane** and **Control plane**, while debugging Istio FIRST we need to determine the issue is in data plane or control plane.

## Step-0: check version of istio used

```sh
i version
# client version: 1.12.1
# control plane version: 1.10.4
# data plane version: 1.10.4
```

you can also check the version of envoy used by a specific version of Istio.

```sh
k exec -it <pod-name with in the mesh> -c istio-proxy -- /bin/sh

envoy --version
```

> more you can reach any envoy using curl and perform GET and POST operation. curl localhost:15000/server_info

> version can also be determined by the image deployed in istiod - pilot:1.10.4

## Step-1: Determine the issue with data plane components

Components to debug:

- Envoy sidecar/proxy
- Istio agent

### Debugging Envoy

#### Verify if Envoy sidecar can connect to istiod

Here we trying to validate that connection between the pod and istiod is successful.

**Effect of failure**: if connectivity fails, then the configuration(vs, dr, sidecar ...) that we apply within the cluster will not be sent back to the sidecars from IstioD

```sh
# sync status between istiod and envoy
# this shows, CDS - cluster, LDS - listener, EDS - endpoint and RDS - routes
i proxy-status(ps) [po/pod-name-with-issue]

k exec -it <any-pod-within-the-mesh-with-curl> -- curl -sS istiod.istio-system:15014/debug/endpointz # should return 2XX response
```

#### Enable & analyse envoy access logs for sidecars

- Access logs are spitted out by envoy, and are not implemented in istio.
- Access logs helps diagnosing E2E traffic flow b/w microservices and failures.
- by default - turned off

[refer more about access log](https://istio.io/latest/docs/tasks/observability/logs/access-log/#default-access-log-format)

```sh
# GLOBALLY: enable access logs
## Global mesh option: https://istio.io/latest/docs/reference/config/istio.mesh.v1alpha1/
## They are pre-configured if we are using profiles, or can use --set while installing the cluster like below
i install --set profile=demo --set meshConfig.accessLogEncoding="JSON" # TEXT(default)
i install --set profile=demo --set meshConfig.accessLogFile="/dev/stdout" # blank string will disable access logging
## check if access log is enabled
k get cm istio -n istio-system -o yaml | grep -i accesslog
## can also update the access logs
k edit cm istio -n istio-system
```

```yaml
apiVersion: v1
data:
  mesh: |-
    accessLogFile: /dev/stdout
    accessLogEncoding: JSON
    defaultConfig:
      discoveryAddress: istiod.istio-system.svc:15012
      proxyMetadata: {}
      tracing:
        zipkin:
          address: zipkin.istio-system:9411
    enablePrometheusMerge: f
```

```yaml
# For a NAMESPACE: enable access log(dont use this if access log is already enabled globally, it will duplicate the logs)
## this is done using EnvoyFilter or via Telemetry
### EnvoyFilter
apiVersion: networking.istio.io/v1alpha3
kind: EnvoyFilter
metadata:
  name: access-log
  namespace: default
spec:
  configPatches:
  - applyTo: NETWORK_FILTER
    match:
      context: ANY
      listener:
        filterChain:
          filter:
            name: "envoy.http_connection_manager"
    patch:
      operation: MERGE
      value:
        typed_config:
          "@type": "type.googleapis.com/envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager"
          access_log: ## area to focus on
          - name: envoy.file_access_log
            config:
              path: /dev/stdout
### Telemetry ( 1.12.0)
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: mesh-default
  namespace: istio-system
spec:
  accessLogging:
    - providers:
      - name: envoy
```

```sh
# View access logs
## refer 12a-sample-access-log.json. NOTE: below fields
### downstream is something which connects to Envoy proxy
### upstream is something which envoy connects to(or forwards traffic to).
### response flag: it contains lots of information when error happens(envoy docs describe more about response flag.)
### for other fields refer Envoy documentation: https://www.envoyproxy.io/docs/envoy/latest/configuration/observability/access_log/usage
k logs <pod-with-istio-enabled> -c istio-proxy # TEXT format is very difficult to read.
```

#### Configure & analyse envoy debug logs

Envoy as an application spits out logs, and they are useful in debugging the sidecar we are using within Istio.

By default the log level is WARNING, we need to change it to DEBUG. NOTE: this is expensive as there will be too much logs and is not recommended for production system.

**Usecase**: this is useful when we dont even get to access logs, we can see the sidecar logs. This may popup some mTLS issue

```sh
i pc log <pod> --level debug # no restart required recommended way

## there are differnt types of logger (enable few)
i pc log <pod-name in the mesh> --level upstream:debug,connection:debug
i pc log <pod-name in the mesh> --level router:debug,http:debug,pool:debug,client:debug,connection:debug

# other ways
## add pod annotation: to make change in the sidecar we need to restart the po
k annotate po <po-name> "sidecar.istio.io/logLevel=debug"

## enable globally: this also need pod restart
i install --set profile=demo --set values.global.proxy.logLevel=debug
```

> Refer to 12d-sample-envoy-debug-log.json

#### Envoy configuration dump

*this are internals of Envoy*.

Envoy configurations itself are split into logical components chain(refer: 06-envoy-configuration-overview) are: *each of this configs can be loaded without impacting others dynamically*

- **Listeners**: ip:port | similar to server/socket address | request is processed by envoy only if there is a corresponding listeners on it. If there are multiple matches, the *most specific one* is selected. *There are different type of listener: such as one for binding the the ip:port and other are for matching*
`i pc l pod-name | grep "shared.kafka"`
- **Filter Chains**: listeners binds to filter chains | they can be based on many criteria like destination, ip address etc
- **HTTP Filters**: filter chain then branches to filters | filters are run in order | if filter do not allow the request it can terminate | for example authz can terminate if user do not authenticate
- **TCP Filters**: filter chain then branches to filters | same as above only that its for TCP
- **Routes**: Routes are only used by HTTP filters, so there is a routing table | it specifies which cluster to send the traffic to
- **Clusters**: clusters are alias name for destination servers. Multiple endpoints creates a cluster.
- **Endpoints**: they are the IP address of the destination servers.

```sh
# validate listeners
# look for the service, you are trying to reach as egress or ingress
i proxy-config(pc) listeners(l) po/<pod-name>
```

#### Validate if ROOTCA is identical for pair of pod

```sh
i pc rootca-compare <pod-1> <pod-2>

# OUTPUT
## Both [pod-1] and [pod-2] have the identical ROOTCA, theoretically the connectivity between them is available
```

##### Listener debugging

*look at all the listeners(without -o json), figure out the listener and then look at the configuration by specifying the address and port*.

```sh
# validate virtual inbound listener
## port number is static(implemented by istio): 15006
## this listener listens to all the request that comes from outside the pod
i pc listeners <listener>.<ns>/<podname> --address 0.0.0.0 --port 15006 -o json

# validate virtual outbound listener
## port number is static(implemented by istio): 15001
## this listener listens to all the request that comes within the pod
## this is only used where there is no specific outbound listener
i pc listeners <listener>.<ns>/<podname> --address 0.0.0.0 --port 15001 -o json

# outbound listeners
i pc listeners <listener>.<ns>/<podname> --address 0.0.0.0 --port 80 -o json

# all the listeners
i pc listeners <listener>.<ns>/<podname> -o json
```

> ExampleL refer example-1 in 12b-debugging-istio.md

### Istio agents

They are part of sidecar and runs inside the same container as envoy. They are sync and other function between istiod and envoy.

#### check your cluster configuration

```sh
i analyze -A
```

#### debug Mutual TLS in Istio

There are two resources which impact Mutual TLS

- Destination Rule
  - determines what type of traffic is sent
  - modes:
    - DISABLE: sends plaintext, *common for services outside the mesh*
    - ISTIO_MUTUAL: sends mTLS
    - SIMPLE/MUTUAL: used to originate TLS *in this certificate management is done by users*

- PeerAuthentication
  - determines what type of traffic is accepted
  - modes:
    - DISABLE: accepts plain text
    - STRICT: accept only mTLS
    - PERMISSIVE: accepts mTLS or plainText

**Auto mTLS**: a feature what was added in 1.6 sometime around mid 2020. *this was implemented mainly because of migration difficulties*

- Auto mTLS is a helm option that is passed when installing Istio
- To turn on-off we need to re-deploy istio, which may break your cluster traffic.
- this feature is deprecated and made default

This is how it works:

- if there is a DR then that is considered for communication, irrespective of PeerAuthentication. *for example if peerauthentication is STRICT and destinationrule is DISABLED, traffic is sent in plainText*
- if the server has a sidecar and peerauthentication allows mTLS, then send mTLS
- Otherwise send plainText

> NOTE: Client and Server in this context are two pods which are trying to communicate.

## Step-2: Determine the issue with control plane components

Determine if the issue with IstioD itself.

### Istiod

#### Validate with dashboard(debug istiod)

Things to look for:

- env variable associated with the process
- memory information (basically this is based on go processes)
- CLI arguments
- Logging scope - debug, info, warning, error, fatal and none

```sh
i dashboard controlz istiod-8479c65c9d-52p84 -n istio-system
# UI should open in your default browser
```

#### Istiod Metrics

Grafana >> search for istio control-plane and look for below useful matrices

- Total number of invalid config: **pilot_total_xds_rejects** > *rejections generally indicate misconfigurations or bugs*
- Time to push config update to proxy/sidecars: **pilot_proxy_convergence_time** > *slowertime may lead to delays in config or endpoints updates. This occurs mostly if control plane is under-scaled*.
- Number of XSD clients: **pilot_xds** > *useful to spot unbalanced load distribution*
