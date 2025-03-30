# Flux

This example demonstrates how to leverage a [Flux](https://fluxcd.io) based OCI artifact within Argo CD. If an existing Flux based artifact does not already exist, steps will be included to produce an OCI artifact for use within this exercise.

## Flux OCI Support

Flux includes support for consuming OCI artifacts containing Kubernetes resources. In addition, the [Flux CLI](https://fluxcd.io/flux/cmd) can be used to package and publish content that can be consumed natively by Flux and the [Source Controller](https://fluxcd.io/flux/components/source). Content can either be packaged using the aforementioned CLI or conform to the packaging requirements where content is stored using the following MeidaTypes:

* Artifact - `application/vnd.oci.image.manifest.v1+json`
* Config - `application/vnd.cncf.flux.config.v1+json`
* Content - `application/vnd.cncf.flux.content.v1.tar+gzip`

## Prerequisites

Ensure that you have the [prerequisite tooling](../README.md#tools) installed on your machine. In addition, if an Flux based OCI artifact will be produced as described within this guide, follow the steps to install and configure the [Flux CLI](https://fluxcd.io/flux/cmd).

Next, set the location where the OCI image will be published as an environment variable called `REGISTRY_REPOSITORY_URL`. For example, if the destination for the image is quay.io/myuser/helm:latest, the value of `REGISTRY_REPOSITORY_URL` would be `quay.io/myuser`

```shell
export REGISTRY_REPOSITORY_URL=<registry_repository_url>
```

Finally, export the namespace Argo CD is deployed within:

```shell
export ARGOCD_NAMESPACE=<namespace>
```

These variables will be referenced within each of the following sections in this example.

## Publish a Flux based OCI Artifact

Using the Flux CLI, publish the contents of the [manifests](manifests) directory containing a `ConfigMap` to the registry:

```script
flux push artifact oci://${REGISTRY_REPOSITORY_URL}/flux:latest \
    --path="./manifests" \
    --revision="$(git branch --show-current)" \
    --source="https://github.com/sabre1041/argocd-oci-examples"
```

## Create the Argo CD Application

Create the Argo CD _Application_ which will retrieve the OCI image and deploy the contents within a namespace called `flux`

```shell
envsubst < applications/flux-application.yaml | kubectl apply -f-
```

By inspecting the newly created `Application`, the `status` displays a message a manner similar to the [Custom Media Type](../custom-media-type/README.md) exercise since the Flux specific MediaType is not supported, by default, in Argo CD. 

Add the Flux MediaType `application/vnd.cncf.flux.content.v1.tar+gzip` which is used for the layer content to the default value of the  `ARGOCD_REPO_SERVER_OCI_LAYER_MEDIA_TYPES` environment variable within the Argo CD Repository Server as shown below:

```text
application/vnd.oci.image.layer.v1.tar;application/vnd.oci.image.layer.v1.tar+gzip;application/vnd.cncf.helm.chart.content.v1.tar+gzip;application/vnd.cncf.flux.content.v1.tar+gzip
```

Once the Argo CD Repository Server restarts, synchronize the _Application_ and confirm that the _ConfigMap_ called `flux` has been created within the `flux` namespace.

```
kubectl get configmap -n flux flux
```
