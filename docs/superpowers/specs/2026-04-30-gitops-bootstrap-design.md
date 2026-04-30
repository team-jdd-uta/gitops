# GitOps Bootstrap Design

## Scope

Bootstrap a `dev` cluster with Argo CD using an `app-of-apps` structure.

## Structure

- `bootstrap/root-application.yaml` is the only required entrypoint manifest.
- `bootstrap/root/` contains the Kustomize bundle that the root app syncs.
- `bootstrap/root/applications/` contains child `Application` resources for:
  - `external-secrets`
  - `external-dns`
  - `backend`
  - `frontend`
- `bootstrap/repositories/` contains example Argo CD repository credential
  manifests with placeholders.
- `clusters/dev/` contains environment notes and namespace manifests.

## Assumptions

- Argo CD is already installed in the target cluster.
- This repository and the external app repositories may require credentials.
- `backend` and `frontend` are deployed from external Git repositories.

## Required inputs

- GitOps repository URL
- Backend repository URL, revision, and manifests path
- Frontend repository URL, revision, and manifests path
- External DNS Route53 domain filter, TXT owner ID, and IRSA role ARN
- Any repository credentials needed by Argo CD
