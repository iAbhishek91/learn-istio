# Circuit Breaking

One of difficult challenges of distributed architecture or microservice is **Cacading failure**(failure of one part will trigger failure to another part in the architecture).

- Cacading failure are very unpredictable.
- they are difficult to identify the area of problem. time taking.

Example of cascading failure:

In busy microservice architecture, one **microservice-C** to **microservice-D** is taking way more long response time, and few requests are timing out. Now, massive amount of requets are stacked at **microservice-C** and due to failure of **microservice-D**, **microservice-C** behave unexpectedly, or probably fail. This may impact the entire systems in chain to hault.

Solution to avoid cascading failure is **Circuit Breaking**

## How circut breaking software works

Circuit breaking software are configurable to watch the network traffic and allow network to flow. In case there are lots of traffic/requests waiting the circuit breaking software will pause(for example returning 503 even without doing anything) that traffic.

It also checks occationally, the health of the microservice and allow the traffic to flow.

This will help to stop cascading the failure. As well help the struggling microservice to recover(probably auto scaling or something like that)

## Other circuit breaking softwares

- Hystrix: system created by netflix(is not in active development). However we need to have a circuit breakier logic built in within the microservices.

## Istio implementation of circuit breaking

Envoy proxy sidecar implements circuit breaking.
