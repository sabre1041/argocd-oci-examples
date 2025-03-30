# Authentication

This example demonstrates how to use [ORAS](https://oras.land) to package and publish an OCI image to an authenticated registry and consume the resources within Argo CD.

## Prerequisites

Ensure that you have the [prerequisite tooling](../README.md#tools) installed on your machine.

Next, set the location where the OCI image will be published as an environment variable called `AUTHENTICATED_REGISTRY_REPOSITORY_URL`. For example, if the destination for the image is quay.io/myuser/authentication:latest, the value of `AUTHENTICATED_REGISTRY_REPOSITORY_URL` would be `quay.io/myuser`

```shell
export AUTHENTICATED_REGISTRY_REPOSITORY_URL=<authenticated-registry_repository_url>
```

Finally, export the namespace Argo CD is deployed within:

```shell
export ARGOCD_NAMESPACE=<namespace>
```

These variables will be referenced within each of the following sections in this example.

## Publish the OCI Artifact

Publish the `ConfigMap` resource located within the [manifests](manifests) directory using ORAS. Ensure that you have authenticated against the remote registry first:

```script
oras login $(echo "${REGISTRY_REPOSITORY_URL%/*}")
```

Once authenticated, push the artifact to the registry

```script
pushd manifests
oras push ${REGISTRY_REPOSITORY_URL}/authentication:latest .
popd
```

## Authenticate to the Registry

While registry credentials can be created delaratively, utilize the Argo CD UI to specify the credentials to the authenticated registry.

Navigate to the Argo CD UI and once authenticated, select **Settings** from the left hand navigation bar.

Select **Connect Repo** and enter the following into the dialog:

Connection Method: HTTP/HTTPS
Type: oci
Project: default
Repository URL: ${AUTHENTICATED_REGISTRY_REPOSITORY_URL}/authentication
Username: `<username>`
Password: `<password>`

Be sure to substitute the value of the `AUTHENTICATED_REGISTRY_REPOSITORY_URL` environment variable along with the registry username and password.

Click **Connect** to create the Repository.

A green checkmark should be displayed next to the newly created repository indicating a successful connection.

Alternatively, repository credentials can be provided declaratively by creating a secret similar to the following:

```yaml
kind: Secret
apiVersion: v1
metadata:
  name: authenticated-repo
  namespace: ${ARGOCD_NAMESPACE}
  labels:
    argocd.argoproj.io/secret-type: repository
stringData:
  password: ${AUTHENTICATED_REGISTRY_REPOSITORY_PASSWORD}
  project: default
  type: oci
  url: oci://${AUTHENTICATED_REGISTRY_REPOSITORY_URL}/authentication
  username: ${AUTHENTICATED_REGISTRY_REPOSITORY_USERNAME}
type: Opaque
```

## Create the Argo CD Application

Create the Argo CD _Application_ which will retrieve the OCI image and deploy the contents within a namespace called `authentication`

```shell
envsubst < applications/authentication-application.yaml | kubectl apply -f-
```

Confirm the _ConfigMap_ called `authentication` has been created within the `authentication` namespace.

```
kubectl get configmap -n authentication authentication
```
