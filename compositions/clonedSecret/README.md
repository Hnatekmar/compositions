# ClonedSecret

Is composition that clones one secret to another using crossplane kubernetes provider. This is usefull for example for transfering secrets between multiple clusters

NOTE: this requires <https://marketplace.upbound.io/providers/crossplane-contrib/provider-kubernetes/v1.2.0> to be present in cluster

# Usage

Lets say you have secret `token` in your local cluster and wish to clone it to remote cluster as secret `cloned-token`

```yaml
apiVersion: hnatekmar.xyz/v1alpha1
kind: ClonedSecret
metadata:
  name: clone
spec:
  src:
    providerConfigRef: # name of k
      name: local
    name: token
    namespace: default
  dst:
    providerConfigRef:
      name: remote
    name: cloned-token
    namespace: default
```
