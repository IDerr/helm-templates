# CloudZero Helm Templates

Curated Helm chart sources and marketplace metadata for CloudZero Apps.

## Layout

- `charts/<app>`: Helm charts installable by per-project ArgoCD.
- `catalog/apps/<app>/<version>.yaml`: immutable CloudZero marketplace app definitions.
- `catalog/index.yaml`: bounded catalog index for the CloudZero API to cache.

## AppDefinition format (canonical)

Every app definition uses the `install`/`edit` fields format — **not** a raw JSON
`schema` block (that older shape is a legacy fallback in the dashboard: it can't
mark fields optional, can't separate install-only fields from editable ones, and
can't map one field onto multiple chart values).

```yaml
spec:
  display: { title, summary, categories }
  source:  { repoURL, path, targetRevision }
  defaults: {}          # chart values applied before field mappings
  install:
    fields:             # rendered as the install form, grouped by `section`
      - key: adminPassword
        type: password  # string | password | boolean | integer | number
        label: Initial admin password
        section: OpenCloud
        default: admin
        required: true
        minLength: 5    # also: maxLength, minimum, maximum, pattern, enum
    values:             # maps form fields -> helm value paths (one or many)
      - from: fields.adminPassword
        to: opencloud.adminPassword
  edit:                 # same shape; only fields that DO something post-install
    fields: []
    values: []
```

Rules of thumb:

- First-boot-only settings (initial passwords, seed users) go in `install` only —
  offering them in `edit` is misleading since changing them does nothing.
- A field that should update several chart values lists them all under `to:`.
- Keep chart-level validation in `charts/<app>/values.schema.json`; the catalog
  definition only describes the marketplace form.

## First App

`hello-world` is the smoke-test app for the marketplace path. It deploys
an unprivileged nginx container serving a static HTML page from a ConfigMap.

## OpenCloud

`opencloud` is the first richer app chart. It follows OpenCloud's
single-container development deployment shape: one OpenCloud container,
config/data PVCs, and optional Gateway API exposure. It deliberately does
not vendor OpenCloud's production stack with Keycloak, PostgreSQL, MinIO,
and office integrations yet.

## Flamenco

`flamenco` deploys Blender's Flamenco Manager with a separately scalable CPU
worker Deployment for render-farm demos. Manager state and shared render storage
are PVC-backed. The chart defaults the shared PVC to ReadWriteOnce and
co-locates workers with the manager; set shared storage to ReadWriteMany when
the cluster has RWX storage and workers should spread more freely.
