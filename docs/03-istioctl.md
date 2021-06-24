# Istio CTL

A command line tool which generates yaml file and optionally apply the manifest in the cluster. However we can download Istio manifest file and apply them manually.

Hence this is optional.

Istio is mostly installed as CRD whether its installed using manifest or Istioctl.

Step-1: initialize Istio, create bunch of CRD and istio-system namespace.
Step-2: Deploy istio (control plane)
Step-3: Deploy kiali secrets(this is optional in newer release)
Step-4: Deploy your application
