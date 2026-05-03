# Project Audit Remediation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Resolve non-auth `PROJECT_AUDIT.md` findings in sequential, mergeable phases.

**Architecture:** Keep responsibilities in their current services. Avoid broad rewrites, keep AI service stateless, keep GitOps as deployment source of truth, and only change runtime/config behavior that directly addresses audit findings.

**Tech Stack:** Node.js/Express, Spring Boot, Vue, FastAPI, Argo CD/Helm, Terraform, Jenkins.

---

## Global Rules

- Pull latest `main` before every issue.
- Work one issue at a time.
- Use branch names like `fix/46-redis-stream-recovery` or `chore/47-frontend-source-of-truth`.
- Use commit messages like `fix: recover pending stream messages`.
- Push, open PR, include `Closes #N`, merge, pull local `main`, delete local branch, then continue.
- Do not add tests or run test suites until the final joint verification pass.
- Do not implement auth/authorization enforcement.

## Phase 1: Runtime and Deployment Correctness

### Issue 1A: Redis Stream Consumer Reliability

**Repository:** `backend-redis-stream-mongo-consumer`

**Scope:**
- Add idempotent Mongo storage by persisting `source_stream_id` from Redis Stream key/id.
- Add pending message recovery using Redis `XAUTOCLAIM` or equivalent node-redis command support.
- Keep existing API behavior unchanged.

**Files likely touched:**
- `src/streams/commentConsumer.js`
- `src/services/commentService.js`
- `src/models/Comment.js`
- `consumer.env.example`

**Deferred verification:**
- No tests now.
- Final pass should include duplicate stream item replay and pending recovery check.

### Issue 1B: Chat Stream Append Observability

**Repository:** `backend-chat-service`

**Scope:**
- Make Redis Stream append failures visible with counters/log context.
- Avoid silent queue drops where possible without blocking every chat message indefinitely.
- Document behavior in README if code-level guarantee remains best-effort.

**Files likely touched:**
- `src/main/java/com/example/chat/service/impl/RedisMessageBrokerService.java`
- README or service docs if present.

**Deferred verification:**
- Final pass should include queue-full/Redis-failure behavior review.

### Issue 1C: Redis SSL Configuration Alignment

**Repositories:** `gitops`, `infra`

**Scope:**
- Align dev GitOps Redis URL/protocol and Spring Redis SSL flags with Terraform ElastiCache transit encryption setting.
- If keeping plaintext Redis in dev, use `redis://` and `SPRING_REDIS_SSL=false` consistently.
- Leave stage/prod hardening for Phase 3 docs unless required by current values.

**Files likely touched:**
- `gitops/bootstrap/root/applications/*.yaml`
- `infra/terraform/environments/dev/global.tfvars.example`
- `infra/terraform/layers/*` only if defaults are inconsistent.

## Phase 2: Source-of-Truth and Repository Hygiene

### Issue 2A: Frontend Runtime Data Source Cleanup

**Repository:** `frontend-ui-vue`

**Scope:**
- Gate fallback/mock stream data behind `VITE_DEMO_MODE=true`.
- Remove committed `.env`; keep `.env.example`.
- Remove or archive duplicated Kubernetes `manifest/` files if GitOps is the source of truth.

**Files likely touched:**
- `src/App.vue`
- `src/components/*`
- `.gitignore`
- `.env.example`
- `manifest/*`

### Issue 2B: GitOps Inactive Application Cleanup

**Repository:** `gitops`

**Scope:**
- Move inactive Application files out of active-looking paths or rename folder to `applications-disabled`.
- Keep root kustomization unchanged except where references are intentionally updated.
- Add a short README note explaining active vs reference apps.

**Files likely touched:**
- `bootstrap/root/applications/*.yaml`
- `bootstrap/root/applications-disabled/*`
- `README.md`

### Issue 2C: RTMP and Kafka CI Source-of-Truth Cleanup

**Repositories:** `backend-rtmp`, `backend-kafka-outbox`

**Scope:**
- RTMP: remove or redirect duplicate root Jenkinsfile so `jenkins/Jenkinsfile` is unambiguous.
- Kafka outbox: either complete Jenkins TODOs enough for reference image checks or clearly mark the repo/pipeline reference-only.

**Files likely touched:**
- `backend-rtmp/Jenkinsfile`
- `backend-rtmp/README.md`
- `backend-kafka-outbox/jenkins/Jenkinsfile`
- `backend-kafka-outbox/README.md`

## Phase 3: Operational Safety and Documentation

### Issue 3A: RTMP Image Pinning

**Repository:** `backend-rtmp`

**Scope:**
- Replace `alfg/nginx-rtmp:latest` with an explicit tag or digest.
- Document update procedure.

### Issue 3B: GitOps Helm Operational Defaults

**Repository:** `gitops`

**Scope:**
- Add optional values for probes/resources/securityContext without breaking existing apps.
- Document route/path mapping and active app list.

### Issue 3C: Infra Documentation and Safer Examples

**Repository:** `infra`

**Scope:**
- Document dev/stage/prod example assumptions.
- Narrow example values where safe without changing actual deployed state unexpectedly.

## Final Joint Verification Pass

Run after all phases are merged:

- Service syntax/build checks.
- Frontend build.
- Terraform validate for touched layers.
- Kustomize/Helm render checks.
- Manual runtime checks for chat, room list, AI summary, Redis stream recovery behavior.
- Update `docs/PROJECT_AUDIT.md` with resolved/residual status.
