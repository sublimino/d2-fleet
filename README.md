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

The content of this repository is packaged as an OCI Artifact, signed and published to GitHub Container Registry
using the [push-artifact](https://github.com/controlplaneio-fluxcd/d2-fleet/blob/main/.github/workflows/push-artifact.yaml)
GitHub Actions workflow.

The Flux Operator running on the Kubernetes clusters in the fleet is configured with a
[FluxInstance](https://github.com/controlplaneio-fluxcd/d2-fleet/blob/main/clusters/staging/flux-system/flux-instance.yaml)
pointing to the OCI Artifact that defines the desired state of each cluster.

The infrastructure components from `d2-infra` and the applications from `d2-apps` follow the same pattern
and are packaged as OCI Artifacts. The delivery of these components is performed by the Flux Operator
using the [ResourceSet](https://github.com/controlplaneio-fluxcd/d2-fleet/tree/main/tenants) definitions.

