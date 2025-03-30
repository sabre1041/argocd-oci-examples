# Archive

This example demonstrates how to use [ORAS](https://oras.land) to take an existing compressed archive containing Kubernetes manifests and package and publish an OCI image to a registry and consume the resources within Argo CD.

## Prerequisites

Ensure that you have the [prerequisite tooling](../README.md#tools) installed on your machine.

Next, set the location where the OCI image will be published as an environment variable called `REGISTRY_REPOSITORY_URL`. For example, if the destination for the image is quay.io/myuser/archive:latest, the value of `REGISTRY_REPOSITORY_URL` would be `quay.io/myuser`

```shell
export REGISTRY_REPOSITORY_URL=<registry_repository_url>
```

Finally, export the namespace Argo CD is deployed within:

```shell
export ARGOCD_NAMESPACE=<namespace>
```

These variables will be referenced within each of the following sections in this example.

## Package Manifests into a Compressed Archive

Argo CD supports OCI images containing a single layer of content. When using ORAS as the tool for packaging and publishing content, the source can either be individual files and folders, or a compressed archive.

Package the contents of the [manifests](manifests) directory which contain a `ConfigMap` as a compressed archive using the following command:

```shell
tar -czvf archive.tar.gz -C manifests .
```

## Publish the OCI Artifact

Publish the archive created within the prior section using ORAS.

```script
oras push ${REGISTRY_REPOSITORY_URL}/archive:latest archive.tar.gz:application/vnd.oci.image.layer.v1.tar+gzip
```

ORAS allows for MediaTypes to be associated to individual resources as it is published. Since the MediaType `application/vnd.oci.image.layer.v1.tar+gzip` is being used, Argo CD will natively be able to resolve the content 

## Create the Argo CD Application

Create the Argo CD _Application_ which will retrieve the OCI image and deploy the contents within a namespace called `archive`

```shell
envsubst < applications/archive-application.yaml | kubectl apply -f-
```

Confirm the _ConfigMap_ called `archive` has been created within the `archive` namespace.

```
kubectl get configmap -n archive archive
```
