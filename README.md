# Compositions

A collection of [Crossplane](https://www.crossplane.io/) compositions designed to solve real-world infrastructure and Kubernetes resource management problems.

This repository provides reusable, pipeline-based compositions that can be installed into any Crossplane-powered control plane to extend its capabilities with opinionated, production-ready resource patterns.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Repository Structure](#repository-structure)
- [Available Compositions](#available-compositions)
  - [ClonedSecret](#clonedsecret)
- [Quick Start](#quick-start)
- [Usage](#usage)
- [Developing New Compositions](#developing-new-compositions)
- [License](#license)

## Prerequisites

Before using any composition from this repository, you need:

- A **Kubernetes cluster** with [Crossplane](https://docs.crossplane.io/latest/software/install/) installed (v1.14+ recommended).
- The **Crossplane CLI** (`crossplane`) for package management and validation.
- Any **provider packages** required by the specific composition (e.g., `provider-kubernetes` for `ClonedSecret`).
- The **composition functions** listed in each composition's `functions.yaml` installed in your Crossplane control plane.

## Repository Structure

```
compositions/
├── <composition-name>/
│   ├── composition.yaml     # Crossplane Composition resource (Pipeline mode)
│   ├── functions.yaml       # Function packages required by this composition
│   ├── xrd.yaml             # CompositeResourceDefinition (XRD) — the API surface
│   └── README.md            # Composition-specific documentation
├── README.md                # This file
└── LICENSE                  # Unlicense (public domain)
```

Each composition lives in its own directory under `compositions/` and is fully self-contained:

- **`composition.yaml`** — Defines the Crossplane Composition using the Pipeline execution mode. Each pipeline step references a Crossplane Function (e.g., `function-kcl`).
- **`functions.yaml`** — Lists the `Function` package resources that must be installed in the cluster before the composition can be used.
- **`xrd.yaml`** — The `CompositeResourceDefinition` that declares the custom API (kind, group, version, and OpenAPI schema) exposed to end users.
- **`README.md`** — Documents the composition's purpose, parameters, and usage examples.

## Available Compositions

### ClonedSecret

Copies a Kubernetes Secret from one cluster to another using the Crossplane Kubernetes provider. Useful for synchronizing secrets (API tokens, registry credentials, TLS certificates, etc.) across clusters without manual intervention or external tooling.

- **API Kind:** `ClonedSecret` (`hnatekmar.xyz/v1alpha1`)
- **Underlying Provider:** `provider-kubernetes`
- **Pipeline Functions:** `function-kcl`, `function-auto-ready`
- **Documentation:** [compositions/clonedSecret/README.md](compositions/clonedSecret/README.md)

## Quick Start

1. **Install Crossplane** on your Kubernetes cluster:
   ```bash
   helm repo add crossplane-stable https://charts.crossplane.io/stable
   helm install crossplane crossplane-stable/crossplane --namespace crossplane-system --create-namespace
   ```

2. **Install the required providers and functions** for the composition you intend to use. For `ClonedSecret`:
   ```bash
   # Install the Kubernetes provider
   cat <<EOF | kubectl apply -f -
   apiVersion: pkg.crossplane.io/v1
   kind: Provider
   metadata:
     name: provider-kubernetes
   spec:
     package: xpkg.upbound.io/crossplane-contrib/provider-kubernetes:v1.2.0
   EOF

   # Install composition functions
   kubectl apply -f compositions/clonedSecret/functions.yaml
   ```

   > **Note:** Before applying the functions.yaml, ensure your cluster can reach the required package registries (e.g., `xpkg.upbound.io`). If your cluster is air-gapped, uses a mirror registry, or needs custom pull secrets, you may need to configure registry mirroring or `packagePullSecrets` in the Crossplane ControllerConfig before the functions will become healthy.

3. **Apply the Composition and XRD:**
   ```bash
   kubectl apply -f compositions/clonedSecret/xrd.yaml
   kubectl apply -f compositions/clonedSecret/composition.yaml
   ```

4. **Wait for all packages to become healthy:**
   ```bash
   kubectl get provider,function,providerrevision,functionrevision
   ```

5. **Create a claim using the composition** — see the composition-specific README for the exact resource YAML.

## Usage

Each composition defines a custom Kubernetes resource (the *claim* or *composite resource*) that you create to trigger the composition pipeline. Refer to the individual composition README files for:

- The full resource schema (required and optional fields).
- Provider configuration requirements (source and destination clusters).
- Concrete examples with `kubectl apply`.

## Developing New Compositions

Contributions are welcome! To add a new composition:

1. Create a new directory under `compositions/<your-composition-name>/`.
2. Write the **XRD** (`xrd.yaml`) defining the composite resource API.
3. Write the **Composition** (`composition.yaml`) using the Pipeline mode.
4. Declare required **Functions** in `functions.yaml`.
5. Document the composition in a `README.md` following the existing style.

See the [ClonedSecret](compositions/clonedSecret/) composition as a reference implementation.

## License

This project is dedicated to the public domain under the [Unlicense](LICENSE). You are free to use, modify, and distribute the code without restriction.
