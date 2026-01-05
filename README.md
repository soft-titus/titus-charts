# Titus Charts

This repository contains Helm charts for
deploying applications and services on Kubernetes.

The primary focus of this repository is to provide reusable,
opinionated, and GitOps-friendly Helm charts that can be consumed
directly or via a Helm chart repository.

------------------------------------------------------------------------

## Available Charts

### Microservice

A reusable Helm chart for deploying a generic microservice on
Kubernetes.

-   Designed for flexibility and safe defaults
-   Suitable for GitOps workflows (e.g. Flux)
-   Supports common Kubernetes primitives such as Deployments, Services,
    Ingress, HPA, PVC, and more

Chart documentation:
See `charts/microservice/README.md` for full usage instructions and
configuration options.

------------------------------------------------------------------------

## Using This Repository as a Helm Repo

Add the Helm repository:

``` bash
helm repo add titus-charts http://soft-titus.github.io/titus-charts
helm repo update
```

List available charts:

``` bash
helm search repo titus-charts
```

Install a chart:

``` bash
helm install demo titus-charts/microservice
```
