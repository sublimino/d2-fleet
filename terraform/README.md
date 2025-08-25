# Bootstrap Flux with Terraform

This example demonstrates how to deploy Flux on a Kubernetes cluster using Terraform
and the `flux-operator` and `flux-instance` Helm charts.

## Usage

Create a Kubernetes cluster using KinD:

```shell
kind create cluster --name flux-staging
```

Install the Flux Operator and deploy the Flux instance on the staging cluster
set as the default context in the `~/.kube/config` file:

```shell
terraform apply \
  -var oci_token="${GITHUB_TOKEN}" \
  -var oci_url="oci://ghcr.io/sublimino/d2-fleet" \
  -var oci_tag="latest" \
  -var oci_path="clusters/staging"
```

Note that the `GITHUB_TOKEN` env var must be set to a GitHub personal access token.
The `oci_token` variable is used to create a Kubernetes image pull secret in the
`flux-system` namespace for Flux to authenticate with the GitHub Container Registry.

Verify the Flux components are running:

```shell
kubectl -n flux-system get pods
```

Wait for the bootstrap process to complete and the Flux instance to sync the cluster state:

```shell
flux get all -A
```

Verify the Flux instance state:

```shell
kubectl -n flux-system get fluxreport/flux -o yaml
```

The output should show the sync status:

```yaml
apiVersion: fluxcd.controlplane.io/v1
kind: FluxReport
metadata:
  name: flux
  namespace: flux-system
spec:
  # Distribution status omitted for brevity
  sync:
    id: kustomization/flux-system
    path: clusters/staging
    ready: true
    source: oci://ghcr.io/sublimino/d2-fleet
    status: 'Applied revision: latest@sha256:b66a51......'
```
