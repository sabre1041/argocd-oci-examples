# Annotations

This example demonstrates how use OCI annotations to decorate an OCI image to expose additional metadata within the Argo CD UI.

The following annotations are supported:

* `org.opencontainers.image.title`
* `org.opencontainers.image.description`
* `org.opencontainers.image.version`
* `org.opencontainers.image.revision`
* `org.opencontainers.image.url`
* `org.opencontainers.image.source`
* `org.opencontainers.image.authors`
* `org.opencontainers.image.created`

## Prerequisites

Ensure that you have the [prerequisite tooling](../README.md#tools) installed on your machine.

Next, set the location where the OCI image will be published as an environment variable called `REGISTRY_REPOSITORY_URL`. For example, if the destination for the image is quay.io/myuser/annotations:latest, the value of `REGISTRY_REPOSITORY_URL` would be `quay.io/myuser`

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
oras push -a "org.opencontainers.image.authors=Andrew Block" \
          -a "org.opencontainers.image.source=https://github.com/sabre1041/argocd-oci-examples" \
          -a "org.opencontainers.image.description=Argo CD OCI Annotations Example" \
          ${REGISTRY_REPOSITORY_URL}/annotations:latest .
popd
```

## Create the Argo CD Application

Create the Argo CD _Application_ which will retrieve the OCI image and deploy the contents within a namespace called `basic-artifact`

```shell
envsubst < applications/annotations-application.yaml | kubectl apply -f-
```

Confirm the _ConfigMap_ called `annotations` has been created within the `annotations` namespace.

```
kubectl get configmap -n annotations annotations
```

## View Annotations within the Argo CD UI

Navigate to the Argo CD UI and select the `annotations` Application.

Select the _History and Rollback_ button where some of the properties that were added to the OCI image as annotations, such as _Maintainers_ and _Description_, are exposed.

