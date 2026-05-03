# Broadcaster Chat Summary Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a broadcaster-triggered chat trend summary flow for the current room using recent chat messages.

**Architecture:** The frontend exposes a summary button only to the room broadcaster. The chat-history consumer owns the manual summary API, validates the requester against room-service, reads MongoDB comments from the last five minutes with a minimum of 10 and maximum of 100 messages, calls AI-chat-summery, stores the summary, and returns it. GitOps and Infra add deploy/CI wiring for the AI summary service.

**Tech Stack:** Vue 3, Node.js/Express/Mongoose, FastAPI/OpenAI SDK, Argo CD Helm app-of-apps, Terraform Jenkins/ECR configuration.

---

### Task 1: AI Trend Summary Service

**Files:**
- Modify: `AI-chat-summery/server/messageService.py`
- Modify: `AI-chat-summery/server/main.py`
- Modify: `AI-chat-summery/server/Dockerfile`
- Create: `AI-chat-summery/jenkins/Jenkinsfile`
- Modify: `AI-chat-summery/README.md`

- [ ] Update prompt from generic sentiment summary to broadcaster-facing trend analysis.
- [ ] Keep response contract `{summary, messageCount}` so the consumer client remains compatible.
- [ ] Add production uvicorn command in Dockerfile without reload.
- [ ] Add Jenkins pipeline based on existing backend pipelines.
- [ ] Keep validation manual for now; do not add tests until final joint verification.

### Task 2: Manual Summary API

**Files:**
- Modify: `backend-redis-stream-mongo-consumer/src/app.js`
- Modify: `backend-redis-stream-mongo-consumer/src/services/commentService.js`
- Create: `backend-redis-stream-mongo-consumer/src/services/manualSummaryService.js`
- Create: `backend-redis-stream-mongo-consumer/src/client/roomServiceClient.js`
- Modify: `backend-redis-stream-mongo-consumer/src/streams/commentConsumer.js`
- Modify: `backend-redis-stream-mongo-consumer/consumer.env.example`

- [ ] Add `POST /api/chat-history/rooms/:roomId/summary`.
- [ ] Accept requester user id through `X-User-Id` or body `requesterUserId`.
- [ ] Query room-service and require `room.broadcasterId === requesterUserId`.
- [ ] Query comments where `createdAt >= now - 5 minutes`, sorted newest first, limited to 100.
- [ ] Return `409 INSUFFICIENT_CHAT_MESSAGES` when fewer than 10 messages exist.
- [ ] Save successful manual summaries to MongoDB.
- [ ] Disable automatic summaries by default using `SUMMARY_AUTO_ENABLED=false`.

### Task 3: Frontend Broadcaster Button

**Files:**
- Modify: `frontend-ui-vue/src/components/StreamingVideoSection.vue`
- Modify: `frontend-ui-vue/README.md`

- [ ] Show summary button only when `isBroadcaster` is true.
- [ ] Call the manual summary API with `X-User-Id`.
- [ ] Render loading, insufficient-message, error, and success states.
- [ ] Update pinned summary text to match the manual 5-minute trend workflow.

### Task 4: GitOps Deployment

**Files:**
- Create: `gitops/bootstrap/root/applications/ai-chat-summary.yaml`
- Modify: `gitops/bootstrap/root/kustomization.yaml`
- Modify: `gitops/bootstrap/root/applications/backend-redis-stream-mongo-consumer.yaml`
- Modify: `gitops/README.md`

- [ ] Add `ai-chat-summary-dev` Argo CD Application using the shared backend Helm chart.
- [ ] Wire `OPENAI_API_KEY` through ExternalSecret from `team9-mini/dev/backend/ai-chat-summary`.
- [ ] Point consumer `SUMMARY_SERVICE_URL` at `http://ai-chat-summary:8000`.
- [ ] Set `SUMMARY_AUTO_ENABLED=false`.

### Task 5: Infra CI/CD

**Files:**
- Modify: `infra/terraform/environments/dev/global.tfvars.example`
- Modify: `infra/terraform/layers/08-cicd/variables.tf`

- [ ] Add `team9-ai-chat-summary` ECR repository.
- [ ] Add `ai-chat-summary-dev` Jenkins job entry for `AI-chat-summery`.
- [ ] Keep Secret Manager creation for `OPENAI_API_KEY` manual/operator-managed unless a later task decides to manage third-party API secrets through Terraform.

### Manual Verification Later

Run these with the user after implementation:

```bash
python3 -m py_compile AI-chat-summery/server/*.py
npm --prefix backend-redis-stream-mongo-consumer run check
npm --prefix frontend-ui-vue run build
helm template ai-chat-summary gitops/apps/backend/dev -f /tmp/ai-values.yaml
terraform -chdir=infra/terraform/layers/08-cicd fmt -check
```
