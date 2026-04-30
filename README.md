# gitops

This repository bootstraps Argo CD for the `dev` cluster with an
`app-of-apps` structure.

## Bootstrap flow

1. Make sure Argo CD is already installed in the target cluster.
2. Register this gitops repository in Argo CD, or apply the example repo
   credential secret with real values first.
3. Apply the root application:

```bash
kubectl apply -f bootstrap/root-application.yaml
```

4. Argo CD will sync the child Applications declared under
   `bootstrap/root/applications/`.

## What gets bootstrapped

- `external-secrets`
- `external-dns`
- `backend`
- `frontend`

## Files that need real input before sync succeeds

- `bootstrap/root-application.yaml`
  - `spec.source.repoURL`
- `bootstrap/repositories/repository-team9-gitops.example-secret.yaml`
  - gitops repo URL and credentials if the repo is private
- `bootstrap/repositories/repository-backend.example-secret.yaml`
  - backend repo URL and credentials
- `bootstrap/repositories/repository-frontend.example-secret.yaml`
  - frontend repo URL and credentials
- `bootstrap/root/applications/external-dns.yaml`
  - AWS Route53 domain filter
  - IRSA role ARN for `external-dns`
  - TXT owner ID
- `bootstrap/root/applications/backend.yaml`
  - backend repo URL
  - target revision
  - manifests path
- `bootstrap/root/applications/frontend.yaml`
  - frontend repo URL
  - target revision
  - manifests path
- `clusters/dev/bootstrap-info-configmap.yaml`
  - example values checklist for the `dev` cluster

## Notes

- The child `backend` and `frontend` deployments intentionally point to
  external repositories.
- The example repository secrets are not included by default in the root sync
  path. Fill them in and apply them, or register the repositories in Argo CD
  through another secure workflow.

## Real Value Checklist

Replace these example values before the first real sync.

### GitOps repo access

- `bootstrap/root-application.yaml`
  - `spec.source.repoURL`
- `bootstrap/repositories/repository-team9-gitops.example-secret.yaml`
  - `stringData.url`
  - `stringData.username`
  - `stringData.password`

### Backend app source

- `bootstrap/root/applications/backend.yaml`
  - `spec.source.repoURL`
  - `spec.source.targetRevision`
  - `spec.source.path`
- `bootstrap/repositories/repository-backend.example-secret.yaml`
  - `stringData.url`
  - `stringData.username`
  - `stringData.password`

### Frontend app source

- `bootstrap/root/applications/frontend.yaml`
  - `spec.source.repoURL`
  - `spec.source.targetRevision`
  - `spec.source.path`
- `bootstrap/repositories/repository-frontend.example-secret.yaml`
  - `stringData.url`
  - `stringData.username`
  - `stringData.password`

### External DNS

- `bootstrap/root/applications/external-dns.yaml`
  - `domainFilters`
  - `txtOwnerId`
  - `eks.amazonaws.com/role-arn`

### Cluster checklist record

- `clusters/dev/bootstrap-info-configmap.yaml`
  - Replace every example repo URL and domain value with the final dev values.

## Apply Order

Use this order when bringing up the first `dev` cluster bootstrap.

1. Install Argo CD in the target cluster.
2. If the gitops repo is private, copy one of the example repository secrets,
   replace the placeholder values, and apply it to `argocd`.
3. If the backend and frontend repos are private, do the same for their
   repository secrets or register them directly in Argo CD.
4. Replace the example values listed in the checklist above.
5. Optionally apply the namespace bundle:

```bash
kubectl apply -k clusters/dev
```

6. Apply the root app:

```bash
kubectl apply -f bootstrap/root-application.yaml
```

7. Verify in Argo CD that:
   - `team9-root-dev` syncs cleanly
   - `external-secrets-dev`, `external-dns-dev`, `backend-dev`, and
     `frontend-dev` are created
   - no child app is stuck due to repo credential or path errors
