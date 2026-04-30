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
