# ClonedSecret

A Crossplane composition that clones a Kubernetes Secret from one cluster to another using the Crossplane Kubernetes provider. This is useful for transferring secrets (API tokens, registry credentials, TLS certificates, etc.) between multiple clusters without manual intervention.

**NOTE:** This composition requires the [crossplane-contrib/provider-kubernetes](https://marketplace.upbound.io/providers/crossplane-contrib/provider-kubernetes/v1.2.0) package to be installed in your Crossplane control plane.

## How It Works

The composition runs a Crossplane pipeline with three steps:

1. **Observe** the source secret on the source cluster using the `kubernetes.crossplane.io/v1alpha2/Object` kind with the `Observe` management policy.
2. **Create** a new secret on the destination cluster, copying the data from the observed source secret.
3. **Auto-ready** — marks the composite resource as ready once the destination secret is available.

## Usage

Suppose you have a secret named `token` in your local cluster and you want to clone it to a remote cluster as `cloned-token`:

```yaml
apiVersion: hnatekmar.xyz/v1alpha1
kind: ClonedSecret
metadata:
  name: clone
spec:
  # Kubernetes provider config pointing to the source cluster
  src:
    providerConfigRef:
      name: local
    # Name of the secret to copy
    name: token
    # Namespace of the source secret
    namespace: default
  # Kubernetes provider config pointing to the destination cluster
  dst:
    providerConfigRef:
      name: remote
    # Name of the secret to create in the destination cluster
    name: cloned-token
    # Namespace where the destination secret will be created
    namespace: default
```

## Parameters

### `spec.src`

| Field              | Type   | Required | Description                                   |
|--------------------|--------|----------|-----------------------------------------------|
| `providerConfigRef.name` | string | ✅       | Name of the Kubernetes ProviderConfig that points to the source cluster. |
| `name`             | string | ✅       | Name of the existing Kubernetes Secret to clone. |
| `namespace`        | string | ✅       | Namespace where the source Secret exists.      |

### `spec.dst`

| Field              | Type   | Required | Description                                   |
|--------------------|--------|----------|-----------------------------------------------|
| `providerConfigRef.name` | string | ✅       | Name of the Kubernetes ProviderConfig that points to the destination cluster. |
| `name`             | string | ✅       | Name for the new Secret in the destination cluster. |
| `namespace`        | string | ✅       | Namespace where the new Secret will be created. |

## Prerequisites

Before using this composition, ensure the following resources exist in your Crossplane control plane:

- **Provider:** `crossplane-contrib/provider-kubernetes` v1.2.0+
- **Functions:** `function-kcl` and `function-auto-ready` (as declared in [functions.yaml](functions.yaml))
- **ProviderConfigs:** Two `ProviderConfig` resources — one for the source cluster and one for the destination cluster — each configured with the appropriate kubeconfig or credentials.

## Notes

- The composition copies **all data** from the source secret to the destination secret. Use with caution if the source secret contains sensitive information that should not leave its cluster boundary.
- The destination secret is created with the same `data` (base64-encoded values) as the source. If the source secret is updated, the destination will be reconciled to match on the next Crossplane reconciliation cycle.
- Deleting the `ClonedSecret` claim will **not** delete the destination secret unless the appropriate Crossplane deletion policy is configured.
