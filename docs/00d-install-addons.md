# Istio integration with other softwares

Istio have removed(from 1.6 onwards) softwares like **kiali**, **Jaeger**, and others from its code base and are managed outside Istio.

*That means that we really CANT install all the above softwares using IstioOperator or Istioctl*.

To install the addons just download the yaml and apply that to your cluster.

[Supported istio integration](https://istio.io/latest/docs/ops/integrations/)
