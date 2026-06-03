# CloudZero Helm Templates

Curated Helm chart sources and marketplace metadata for CloudZero Apps.

## Layout

- `charts/<app>`: Helm charts installable by per-project ArgoCD.
- `catalog/apps/<app>/<version>.yaml`: immutable CloudZero marketplace app definitions.
- `catalog/index.yaml`: bounded catalog index for the CloudZero API to cache.

## First App

`hello-world` is the smoke-test app for the marketplace path. It deploys
an unprivileged nginx container serving a static HTML page from a ConfigMap.
