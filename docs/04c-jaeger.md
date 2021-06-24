# Jaeger

Another UI for debugging issues, and part of Istio Control plane.

Jaeger is used for **distributed tracing**. do a deep dive on the services. Kiali gives a very high level view, and this is where jaeger comes into picture.

Using Jaeger, we can track in each network request, detail chain of events occured etc.

There is a effort in standardizing tracing frameworks: "opentracing.io"

Along with many other, Jaeger supports open tracing.

Jaeger is created by Uber, and ZIPKIN another one is created by twitter. Both of these are integrated in the underlying layer of Istio.

> NOTE: distributed tracing is nothing to do with Kubernetes and Istio. If we have a minimal Javascript frontend, with a backend, we can download the client library and implement distributed tracing. However what we are looking to abstract distributing tracing from application level to proxy level.

## Find traces

With traces we want to know, what are the hops, what are the microservices it went through and how did it perform.

We select a trace and then view the spans. Number of span are almost double than expected as proxy contianers are also counted by Jaeger.

- First step is to select the service. *This service is one of hop for the network call and for the staring hop always*
- Select the Loopback as "Custom Time Range". Provide the start time or end time based on the information you have about the incident.
- Click on find traces.
- This is more complicated to find the traces. *Now instead of going through the traces, we can select from the histogram above, this is relatively simple as we have the time in the X-axis and the response time.*
- clik on the trace and view whats going on. *service, request uri, response time, etc etc*
- We can also compare multiple traces by selecting all the traces(using the checkbox).

> NOTE to make trace work, we need to propagate few headers, and this is the ONLY application change we need to do in order to do tracing.

## How Jager does tracing of the HTTP calls

- Application developer needs to update the code, in order to pass few header.
- Those headers are used by Jager to co-relate the https request to create the trace.
- The header is **Globally Unique Identifier** GUID. "guid:x-request-id". example: f9cff561-9271-9235-8eaf-61418f72c573. For each span in the trace will have same GUID.
- Now, the above headers is injected by Istio(or Envoy proxy sidecar container).
  - the log to inject the header is: if "guid:x-request-id" header is missing inject one, else ignore.
- However we need to send few headers(along with "guid:x-request-id" header with the same value that was received) "defined by Istio" in order to perform distributed tracing. Otherwise each proxy contianer will create new GUID and distributing tracing system will create many more unrelated traces. [List of all the headers are documented by Istio](https://istio.io/latest/about/faq/#how-to-support-tracing). Frustrating part is you need to implement the same headers in all the microservices in the architecture to make it work properly. THIS IS ALMOST NOT POSSIBLE FOR PROXY ENVOY SIDECAR CONTAINER TO PERFORM THIS.
