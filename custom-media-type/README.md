# Custom Media Types

This example demonstrates how to use [ORAS](https://oras.land) to package and publish an OCI image to a registry using a Custom MediaType and consume the resources within Argo CD.

## Prerequisites

Ensure that you have the [prerequisite tooling](../README.md#tools) installed on your machine.

Next, set the location where the OCI image will be published as an environment variable called `REGISTRY_REPOSITORY_URL`. For example, if the destination for the image is quay.io/myuser/custom-media-type:latest, the value of `REGISTRY_REPOSITORY_URL` would be `quay.io/myuser`

```shell
export REGISTRY_REPOSITORY_URL=<registry_repository_url>
```

Finally, export the namespace Argo CD is deployed within:

```shell
export ARGOCD_NAMESPACE=<namespace>
```

These variables will be referenced within each of the following sections in this example.

## Publish the OCI Artifact using a custom MediaType

Argo CD supports OCI images containing a single layer of content. This layer must also make use of one of the following MediaTypes:

* application/vnd.oci.image.layer.v1.tar
* application/vnd.oci.image.layer.v1.tar+gzip
* application/vnd.cncf.helm.chart.content.v1.tar+gzip

Additional MediaTypes can be accepted by the Repository Server by appending the comma separated list of accepted MediaTypes by either specifying the `--oci-layer-media-types` argument or within the `ARGOCD_REPO_SERVER_OCI_LAYER_MEDIA_TYPES` environment variable (Recommended).

ORAS, by default, will make use of the MediaType `application/vnd.oci.image.layer.v1.tar+gzip` when publishing content. However, end users can specify an alternate MediaType instead.

Publish the `ConfigMap` resource located within the [manifests](manifests) directory using ORAS and specify that the MediaType `application/vnd.cncf.argoproj.argocd.content.v1.tar+gzip`.

```script
pushd manifests
oras push ${REGISTRY_REPOSITORY_URL}/custom-media-type:latest .:application/vnd.cncf.argoproj.argocd.content.v1.tar+gzip
popd
```

Fetch the manifest and inspect the MediaType of the layer to confirm the custom MediaType was applied:

```shell
oras manifest fetch --format=go-template='{{ (index .content.layers 0).mediaType }}' ${REGISTRY_REPOSITORY_URL}/custom-media-type:latest
```

## Create the Argo CD Application

Create the Argo CD _Application_:

```shell
envsubst < applications/custom-media-type-application.yaml | kubectl apply -f-
```

Inspect the newly created Argo CD _Application_.

```
kubectl get application -n ${ARGOCD_NAMESPACE} custom-media-type -o yaml
```

By inspecting the `status` property, you will see the following:

```
message: 'Failed to load target state: failed to generate manifest for source
1 of 1: rpc error: code = Unknown desc = oci layer media type application/vnd.cncf.argoproj.argocd.content.v1.tar+gzip
is not in the list of allowed media types'
```

## Enabling Additional Media Types

To resolve the issues that are being experienced with the `custom-media-type` _Application_, add `application/vnd.cncf.argoproj.argocd.content.v1.tar+gzip` to the default value of the  `ARGOCD_REPO_SERVER_OCI_LAYER_MEDIA_TYPES` environment variable within the Argo CD Repository Server as shown below:

```text
application/vnd.oci.image.layer.v1.tar;application/vnd.oci.image.layer.v1.tar+gzip;application/vnd.cncf.helm.chart.content.v1.tar+gzip;application/vnd.cncf.argoproj.argocd.content.v1.tar+gzip
```

Once the Argo CD Repository Server restarts, synchronize the _Application_ and confirm that the _ConfigMap_ called `custom-media-type` has been created within the `custom-media-type` namespace.

```
kubectl get configmap -n custom-media-type custom-media-type
```
