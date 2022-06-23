# Istio CTL

A command line tool which generates yaml file and optionally apply the manifest in the cluster. However we can download Istio manifest file and apply them manually. Istioctl also helps us to debug service mesh configs and many other stuff

Hence this is optional.

Istio is mostly installed as CRD whether its installed using manifest or Istioctl.

Step-1: initialize Istio, create bunch of CRD and istio-system namespace.
Step-2: Deploy istio (control plane)
Step-3: Deploy kiali secrets(this is optional in newer release)
Step-4: label you namespace with *istio-injection=enabled*
Step-5: Deploy your application

## Installation of istio

```sh
# default installation - will install default profile
i install

# install a specific profile (look for istio profile in 02-terminology)
i install --set profile=demo

# update the built-in profiles and install istio with those
## step-1: download the profile
i profile dump default > default_profile.yaml # or any other profile name (demo)
## step-2: update the profile manually as per requirement
## step-3a: install istio with the tuned/updated profile [using istioctl]
i install -f default_profile.yaml
## step-3b: install istio with the tuned/updated profile [using kubectl]
i manifest generate -f default_profile.yaml > raw_default_profile.yaml # this commands will change the profile dump to a k8s object
k apply -f raw_default_profile.yaml
```

Few important command used while debugging are

```sh
# analyze IstioD configuration and print validation message
i analyze -A

# analyze IstioD via dashboard
i dashboard controlz istiod-8479c65c9d-52p84 -n istio-system

# check the synchronous status of each envoy sidecar in the mesh with IstioD
# information about cluster, listener, endpoint and routes
# all this is SYNCED then itiod control-plane is working fine,
i proxy-status(ps) [po/<pod-name>]

# proxy config more on 12-debugging-istio.md
istioctl proxy-config <clusters|listeners|routes|endpoints|bootstrap|log|secret> <pod-name[.namespace]>
```
