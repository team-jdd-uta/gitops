# Project Audit Remediation Design

## Goal

Apply the actionable items from `docs/PROJECT_AUDIT.md` in small, mergeable phases while excluding authorization/authentication enforcement work and deferring test creation/execution to the final joint verification pass.

## Exclusions

- Cognito JWT validation, API authorization, and userId ownership enforcement.
- Token refresh/revoke/storage hardening.
- New unit/integration/e2e tests during implementation phases.
- Large architecture rewrites such as replacing Redis Pub/Sub or migrating room categories to a fully managed database taxonomy in one step.

## Workflow

Each phase follows the same sequence:

1. Pull latest `main`.
2. Create a GitHub issue.
3. Create a branch named `{tag}/{issue_num}-{summary}`.
4. Implement only that issue scope.
5. Commit with `{tag}: {summary}`.
6. Push, create PR with `Closes #{issue_num}`.
7. Merge PR.
8. Switch local repo back to `main`, pull, and delete the local branch.
9. Start the next issue only after the previous phase is merged locally.

## Phase Model

### Phase 1: Runtime and Deployment Correctness

Fix issues that can break local/dev runtime or make deployed services point at the wrong target.

- `backend-socket-io-gateway`: align local `CHAT_SERVICE_URL` default or document local context path clearly.
- `backend-chat-service`: reduce silent Redis Stream loss by adding observable failure counters/logging and a less lossy publish ordering if feasible without broad rewrite.
- `backend-redis-stream-mongo-consumer`: add Redis Stream idempotency and pending-message recovery primitives if missing.
- `gitops`/`infra`: align Redis SSL settings with current ElastiCache transit encryption mode.

### Phase 2: Source-of-Truth and Repository Hygiene

Remove or isolate files that make operators/developers misunderstand what is active.

- `frontend-ui-vue`: remove production fallback mock behavior unless explicitly enabled by `VITE_DEMO_MODE=true`.
- `frontend-ui-vue`: remove or archive duplicated `manifest/` Kubernetes manifests if GitOps Helm chart is source of truth.
- `frontend-ui-vue`: remove committed `.env` and keep `.env.example`.
- `backend-rtmp`: remove or redirect duplicate root `Jenkinsfile`.
- `gitops`: move inactive Argo CD Application examples to a disabled/reference folder.
- `backend-kafka-outbox`: either complete Jenkins TODOs for reference build use or mark the repo as reference-only.

### Phase 3: Operational Safety and Documentation

Add low-risk operational safeguards and documentation that do not require auth work or test creation.

- `backend-rtmp`: pin base image away from `latest`.
- `gitops`: add active app/path routing documentation.
- `gitops` Helm chart: add optional probes/resources/securityContext defaults where they do not break existing values.
- `infra`: document stage/prod example differences and narrow example defaults where safe.
- Service docs: align README/API contract notes with current context paths and deployment source of truth.

## Testing Policy

During phases, do not create or run tests unless explicitly requested later. Verification commands, builds, and runtime tests are deferred to the final joint pass with the user. Implementation PR descriptions must clearly state that tests were deferred by request.

## Success Criteria

- Each phase lands through merged GitHub PRs.
- No phase includes auth/authorization enforcement.
- No phase introduces new tests or runs test suites before the final joint pass.
- Local repos are returned to clean `main` after each merged issue.
- `docs/PROJECT_AUDIT.md` can be updated after the phases to mark resolved items and residual risks.
