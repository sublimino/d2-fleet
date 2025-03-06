# d2-fleet

> [!NOTE]
> This repository is part of the reference architecture for the
> [ControlPlane Enterprise for Flux CD](https://fluxcd.control-plane.io/).
>
> The `d2` reference architecture comprised of
> [d2-fleet](https://github.com/controlplaneio-fluxcd/d2-fleet),
> [d2-infra](https://github.com/controlplaneio-fluxcd/d2-infra) and
> [d2-apps](https://github.com/controlplaneio-fluxcd/d2-apps)
> is a set of best practices and production-ready examples for using Flux Operator
> and OCI Artifacts to manage the continuous delivery of Kubernetes infrastructure and
> applications on multi-cluster multi-tenant environments.

## Scope and Access Control

This repository is managed by the platform team who are responsible for
the Kubernetes infrastructure and have direct access to the fleet of clusters.

The platform team that manages this repository must have **admin** rights to the `d2-fleet` repository
and **cluster admin** rights to all clusters in the fleet to be able to perform the following tasks:

- Bootstrap Flux Operator with multi-tenancy restrictions on the fleet of clusters.
- Configure the delivery of platform components (defined in [d2-infra repository](https://github.com/controlplaneio-fluxcd/d2-infra)).
- Configure the delivery of applications (defined in [d2-apps repository](https://github.com/controlplaneio-fluxcd/d2-apps)).

## OCI Artifacts

The content of this repository is packaged as an OCI Artifact, signed and published to GitHub Container Registry
using the [push-artifact](https://github.com/controlplaneio-fluxcd/d2-fleet/blob/main/.github/workflows/push-artifact.yaml)
GitHub Actions workflow.

The Flux Operator running on the Kubernetes clusters in the fleet is configured with a
[FluxInstance](https://github.com/controlplaneio-fluxcd/d2-fleet/blob/main/clusters/staging/flux-system/flux-instance.yaml)
pointing to the OCI Artifact that defines the desired state of each cluster.

The infrastructure components from `d2-infra` and the applications from `d2-apps` follow the same pattern
and are packaged as OCI Artifacts. The delivery of these components is performed by the Flux Operator
using the [ResourceSet](https://github.com/controlplaneio-fluxcd/d2-fleet/tree/main/tenants) definitions.


| Artifact                                                    | Git Repository | Envs                                                       |
|-------------------------------------------------------------|----------------|------------------------------------------------------------|
| `oci://ghcr.io/controlplaneio-fluxcd/d2-fleet`              | d2-fleet       | staging / production / GHA image update automation cluster |
| `oci://ghcr.io/controlplaneio-fluxcd/d2-infra/cert-manager` | d2-infra       | staging / production                                       |
| `oci://ghcr.io/controlplaneio-fluxcd/d2-infra/monitoring`   | d2-infra       | staging / production                                       |
| `oci://ghcr.io/controlplaneio-fluxcd/d2-apps/backend`       | d2-apps        | staging / production                                       |
| `oci://ghcr.io/controlplaneio-fluxcd/d2-apps/frontend`      | d2-apps        | staging / production                                       |

## Onboarding platform components

The platform team is responsible for onboarding the platform components defined as Flux HelmReleases in the
[d2-infra repository](https://github.com/controlplaneio-fluxcd/d2-infra) and set the dependencies
between the components.

Platform components are cluster add-ons such as CRDs and their respective controllers,
and are reconciled by Flux as the **cluster admin**.

To onboard a component from the `d2-infra` repository, the platform team must add a
line for the component in the `.github/workflows/push-artifact.yaml` GitHub Actions
workflow file of the `d2-infra` repository:

```yaml
      ...
      matrix:
        component:
          - cert-manager
          - monitoring
```

With this, an OCI Artifact will be published and signed for the new component.

On the `d2-fleet` repository, the platform team must add a new set of inputs for the
`infra` `ResourceSet`:

```yaml
  ...
  inputs:
    - tenant: "cert-manager"
      tag: "${ARTIFACT_TAG}"
      environment: "${ENVIRONMENT}"
    - tenant: "monitoring"
      tag: "${ARTIFACT_TAG}"
      environment: "${ENVIRONMENT}"
```

With this, the set of base resources for a component will now also be created for the new component.
This set includes an `OCIRepository` object that points to the OCI Artifact, and two `Kustomization`
objects consuming the artifact, `infra-controllers` and `infra-configs`, that together configure the
reconciliation of the new component.

The typical structure of the `d2-infra` repository is as follows:

```shell
./components/
├── cert-manager
│   ├── configs
│   │   ├── base
│   │   ├── production
│   │   └── staging
│   └── controllers
│       ├── base
│       ├── production
│       └── staging
└── monitoring
    ├── configs
    │   ├── base
    │   ├── production
    │   └── staging
    └── controllers
        ├── base
        ├── production
        └── staging
```
