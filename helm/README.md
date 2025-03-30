# Helm

This example demonstrates how to use [ORAS](https://oras.land) to package and publish an OCI image containing a Helm chart to a registry so that it can be consumed by Argo CD.

## Prerequisites

Ensure that you have the [prerequisite tooling](../README.md#tools) installed on your machine.

Next, set the location where the OCI image will be published as an environment variable called `REGISTRY_REPOSITORY_URL`. For example, if the destination for the image is quay.io/myuser/helm:latest, the value of `REGISTRY_REPOSITORY_URL` would be `quay.io/myuser`

```shell
export REGISTRY_REPOSITORY_URL=<registry_repository_url>
```

Finally, export the namespace Argo CD is deployed within:

```shell
export ARGOCD_NAMESPACE=<namespace>
```

These variables will be referenced within each of the following sections in this example.

## Publish the OCI Artifact

Publish the Helm chart located within the [charts](charts) directory to the registry:

```script
pushd charts/argocd-oci
oras push ${REGISTRY_REPOSITORY_URL}/helm:latest .
popd
```

## Create the Argo CD Application

Create the Argo CD _Application_ which will retrieve the OCI image and deploy the contents within a namespace called `helm`

```shell
envsubst < applications/helm-application.yaml | kubectl apply -f-
```

Confirm the _ConfigMap_ called `helm` has been created within the `helm` namespace.

```
kubectl get configmap -n helm helm-argocd-oci
```
