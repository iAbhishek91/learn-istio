# Kiali

UI of Isio. From the official documentation, kiali perform below jobs:

- what microservices are part of my istio service mesh
- how are they connected
- how are they performing

For Kiali to work we need to configure secret to login to the UI. something like below

```sh
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: kiali
  namespace: istio-system
  labels:
    app: kiali
data:
  username: YWRtaW4= # admin
  passphrase: YWRtaW4= # admin
```

## Going around Kiali

- documentation (use the question mark icon on the top right corner)
- Left menu
  - Overview: shows all the namespace Istio is monitoring, applicaitons(pods) count and status, as well as traffic
  - Graph:
    - shows the dynamic view of links and status between the pods or services (select the namespace from top dropdown, and the graph: "version app graph").
    - For any symbol please refer to legend at the bottom of the page.
    - Incoming status and Outgoing status
    - Also have layout to change how the graph looks like(fairly simple)
    - double click on the nodes of the graph will change the view and show incoming and outgoing traffic of that node only.
    - right click on the service or pod to get the information about.
    - Response time in the deplay dropdown will work only with "Workload graph"
  - Applications:
    - shows all the pods, pods are referred as applications.
  - Services
    - From kiali, Actions(top-right), we can manage traffic: "create wighted routing", "create matching routing" and "suspend traffic". *Under the hood, a VirtualService & destinationRules is created automatically by kiali*
  - Istio Config
    - shows virtualServices and DestinationRules
  - workloads
    - all the pods behind a services.

> NOTE: important legend: grey arrow indicates that there is no traffice b/w the services. Also if there is NO traffic for long time, the line/arrow between the edge are removed(stale connection are shown).

## Debugging/diagnose using Kiali

- Error or exception issue
  - Easy Easy, navigate to Graph tab, red(5xx) or yellow(4xx) color arrow.
  - check the service entry(refer docs for service entry).
- Performance issue
  - navigate to graph, then click on the Response Codes
  - select "Service Graph" graph.
  - then select what to show in the edge. for eg: "request per seconds.
  - Dive deeper using **jaeger**(another UI for debugging issues, and part of Istio Control plane)
