# team9-mini GitOps

This repository is the Argo CD source of truth for the `team9-mini` dev cluster.
It uses an app-of-apps structure: one root Application creates the Team9 project
and the child Applications for the services.

## Bootstrap Flow

1. Install Argo CD and platform add-ons from the `infra` repository.
2. Register this gitops repository in Argo CD if it is private.
3. Apply the root application:

```bash
kubectl apply -f bootstrap/root-application.yaml
```

4. Argo CD syncs the child Applications declared under `bootstrap/root/applications/`.

## Active Applications

- `team9-root-dev`
- `backend-user-service-dev`
- `backend-login-service-dev`
- `backend-room-service-dev`
- `backend-chat-service-dev`
- `backend-socket-io-gateway-dev`
- `backend-redis-stream-mongo-consumer-dev`
- `frontend-ui-vue-dev`
- `rtmp-dev`

The root kustomization currently includes the service Applications above.
`external-secrets`, `external-dns`, and old `backend-kafka-outbox` manifests are
kept as references but are not part of the active root sync path.

## Repository Layout

```text
bootstrap/
  root-application.yaml
  root/
    project-team9.yaml
    applications/
apps/
  backend/dev/
  frontend/dev/
  rtmp/dev/
clusters/
  dev/
```

## How Deployments Work

- Backend services share the Helm chart in `apps/backend/dev`.
  Each Argo Application passes service-specific Helm values inline.
- The frontend uses the Helm chart in `apps/frontend/dev`.
- RTMP uses raw Kubernetes manifests under `apps/rtmp/dev`.
- All active child Applications point at this same gitops repository and
  `targetRevision: main`.
- Argo CD automated sync is enabled with `prune` and `selfHeal`.

## Runtime Integrations

- External Secrets Operator and `ClusterSecretStore/aws-secretsmanager` are
  installed by Terraform in `infra/terraform/layers/05-platform-addons`.
- Service `ExternalSecret` resources read from AWS Secrets Manager and create
  Kubernetes Secrets such as `backend-room-service-secret`.
- Stakater Reloader is installed by Terraform. Services with
  `secret.reloader.stakater.com/reload` restart automatically when their Secret
  changes.
- Backend services use the shared ALB host `backend.team9.cloud.skala-ai.com`
  with path-based routing.
- Login service uses the dedicated ALB host `login.team9.cloud.skala-ai.com`.
- Frontend uses `front.team9.cloud.skala-ai.com`.
- RTMP uses a public NLB for TCP `1935` ingest and HTTP `80` HLS/stat endpoints.

## Service Notes

- `backend-room-service-dev` reads RDS credentials from
  `team9-mini-dev-db-02-credentials` and `RTMP_CALLBACK_SECRET` from
  `team9-mini/dev/backend/rtmp`.
- `backend-chat-service-dev` and `backend-socket-io-gateway-dev` read Redis
  Pub/Sub connection values from `team9-mini-dev-redis-pubsub`.
- `rtmp-dev` currently runs image
  `881490135253.dkr.ecr.ap-northeast-2.amazonaws.com/team9-rtmp:154d719`.
- RTMP callback traffic targets `backend-room-service` in the `backend`
  namespace through the RTMP image configuration.

## Apply Order

Use this order when bringing up a fresh dev cluster.

1. Apply the required infra layers, especially `05-platform-addons`, `06-data`,
   `08-cicd`, and `09-auth`.
2. If the gitops repo is private, copy one of the example repository secrets,
   replace the placeholder values, and apply it to `argocd`.
3. Optionally apply the namespace bundle:

```bash
kubectl apply -k clusters/dev
```

4. Apply the root app:

```bash
kubectl apply -f bootstrap/root-application.yaml
```

5. Verify in Argo CD that all active child Applications are `Synced` and
   `Healthy`.

## Verification Commands

```bash
kubectl get applications -n argocd
kubectl get deploy,svc,ingress,externalsecret -n backend
kubectl get deploy,svc,pod -n frontend
kubectl get deploy,svc,pod -n rtmp
```
