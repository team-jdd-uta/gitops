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
- `bootstrap/root/applications/external-dns.yaml`
  - AWS Route53 domain filter
  - IRSA role ARN for `external-dns`
  - TXT owner ID
- `bootstrap/root/applications/backend.yaml`
  - target revision
  - manifests path
- `bootstrap/root/applications/frontend.yaml`
  - target revision
  - manifests path
- `apps/backend/dev`
  - backend manifests, Helm values, or Kustomize overlay
- `apps/frontend/dev`
  - frontend manifests, Helm values, or Kustomize overlay
- `clusters/dev/bootstrap-info-configmap.yaml`
  - example values checklist for the `dev` cluster

## Notes

- The child `backend` and `frontend` deployments intentionally point to paths
  inside this same gitops repository.
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
  - `spec.source.targetRevision`
  - `spec.source.path`
- `apps/backend/dev`
  - replace the placeholder manifests with the real backend deployment source

### Frontend app source

- `bootstrap/root/applications/frontend.yaml`
  - `spec.source.targetRevision`
  - `spec.source.path`
- `apps/frontend/dev`
  - replace the placeholder manifests with the real frontend deployment source

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
3. Replace the example values listed in the checklist above.
4. Replace the placeholder manifests under `apps/backend/dev` and
   `apps/frontend/dev` with the real deployment source.
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
