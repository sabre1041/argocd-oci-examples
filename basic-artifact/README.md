# Basic Artifact

This example demonstrates how to use [ORAS](https://oras.land) to package and publish an OCI image to a registry and consume the resources within Argo CD.

## Prerequisites

Ensure that you have the [prerequisite tooling](../README.md#tools) installed on your machine.

Next, set the location where the OCI image will be published as an environment variable called `REGISTRY_REPOSITORY_URL`. For example, if the destination for the image is quay.io/myuser/basic-artifact:latest, the value of `REGISTRY_REPOSITORY_URL` would be `quay.io/myuser`

```shell
export REGISTRY_REPOSITORY_URL=<registry_repository_url>
```

Finally, export the namespace Argo CD is deployed within:

```shell
export ARGOCD_NAMESPACE=<namespace>
```

These variables will be referenced within each of the following sections in this example.

## Publish the OCI Artifact

Publish the `ConfigMap` resource located within the [manifests](manifests) directory using ORAS.

```script
pushd manifests
oras push ${REGISTRY_REPOSITORY_URL}/basic-artifact:latest .
popd
```

## Create the Argo CD Application

Create the Argo CD _Application_ which will retrieve the OCI image and deploy the contents within a namespace called `basic-artifact`

```shell
envsubst < applications/basic-artifact-application.yaml | kubectl apply -f-
```

Confirm the _ConfigMap_ called `basic-artifact` has been created within the `basic-artifact` namespace.

```
kubectl get configmap -n basic-artifact basic-artifact
```
