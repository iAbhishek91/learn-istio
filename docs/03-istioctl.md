# Istio CTL

A command line tool which generates yaml file and optionally apply the manifest in the cluster. However we can download Istio manifest file and apply them manually. Istioctl also helps us to debug service mesh configs and many other stuff

Hence this is optional.

Istio is mostly installed as CRD whether its installed using manifest or Istioctl.

Step-1: initialize Istio, create bunch of CRD and istio-system namespace.
Step-2: Deploy istio (control plane)
Step-3: Deploy kiali secrets(this is optional in newer release)
Step-4: Deploy your application

```sh
# list of routes
istioctl proxy-config route istio-ingressgateway-7c89d84746-rsrq6 -n istio-system 
```
