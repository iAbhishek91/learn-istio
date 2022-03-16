# Canaries

they are way we deploy application safely into kubenretes cluster.

Without any proper solution(only with native k8s) its **bodge**(badly implemented) canary solution below:

- Implement one service and point to two different deployment with same selector.

>NOTE: THE ABOVE SOLUTION IS NOT SUGGESTED. BELOW WE SHOW HOW TO PERFORM CANARY RELEASES USING ISTIO.

## Specific tags

There are few tags that are important for Istio and to visualize in Kiali are

- **app**: for app graph in kiali
- **version**: for version graph in kiali