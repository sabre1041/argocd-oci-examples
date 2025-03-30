# Argo CD OCI Examples

Examples demonstrating the use of the OCI based content as an Application source within Argo CD.

## Overview

This repository contains a series of examples that demonstrate the use of OCI images as an Application Source for Argo CD. An Argo CD instance with OCI support must be in use in order leverage the examples within this repository. An image that is kept fairly up to date can be found at [quay.io/ablock/argocd:oci](https://quay.io/repository/ablock/argocd).

## Tools

The following tools are leveraged for most of the examples contained within this repository. Use the following steps to install and configure them within your own machine

### ORAS

[ORAS](https://oras.land) is a utility that facilitates the management of Artifacts. Instructions for how to install and configure the utility can be found on the project documentation.

### envsubst

`envsubst` is a CLI tool that facilitates the replacement of environment variables and will be used to inject values into templated files.

## Examples

The following table describes the examples that are included within this repository

| Name | Description |
| ---- | ----------- |
| [Annotations](annotations/README.md) | Demonstrate how to decorate an OCI image with Annotations to provide metadata which in turn are exposed to Argo CD |
| [Archive](archive/README.md) | Publish an OCI artifact containing a compressed archive containing Kubernetes manifests |
| [Authentication](authentication/README.md) | Accessing an OCI image stored within a protected registry |
| [Basic Artifact](basic-artifact/README.md) | Simple example demonstrating the basic capabilities of leveraging an OCI image within Argo CD |
| [Custom Media Type](basic-artifact/README.md) | Demonstrates how to apply a custom _MediaType_ to the OCI artifact and consume within Argo CD |
| [Flux](flux/README.md) | Utilize a Flux created OCI artifact within Argo CD |
| [Helm](helm/README.md) | Helm chart stored as a basic OCI image |
