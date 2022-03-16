# IstioOperator

- deploys all the components of Istio
- also accepts configurations for each components

## Yaml Manifest

This is configurable, hence blank installation installes everything with default configuration.

[Read the spec of IstioOperator](https://istio.io/latest/docs/reference/config/istio.operator.v1alpha1/)

## Status and ComponentStatus

*Below are observed state of IstioOperator and are NOT defined*.

**Status** of IstioOperator depends on status of all components its deploying.

For example if one is ERROR and other are HEALTHY, the status is ERROR.

[Full list of IstioOperator status](https://istio.io/latest/docs/reference/config/istio.operator.v1alpha1/#InstallStatus)

**ComponentStatus** (see list of components in `00b-istioComponents.md) is status of each component.

The status is controlled by the IstioOperator.
