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
  # name of kubernetes provider that points to cluster you wicluster you wicluster you wicluster you wish to copy from
  src:
    providerConfigRef:
      name: local
    # name of secret you wish to copy
    name: token
    # nameespace from which you want to source the secret
    namespace: default
  # name of kubernetes provider that points to cluster you wish to copy to
  dst:
    providerConfigRef:
      name: remote
    # name of secret you wish to create in destination cluster
    name: cloned-token
    # namespace where you wish to create the secret
    namespace: default
```
