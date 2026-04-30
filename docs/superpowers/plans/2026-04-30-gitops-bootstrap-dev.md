# GitOps Bootstrap Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a `dev`-only Argo CD bootstrap with a root Application and four child Applications.

**Architecture:** A manually applied root `Application` points Argo CD at a Kustomize bundle inside this repository. That bundle creates an AppProject plus four child Applications, while repository credentials remain example manifests that users fill in separately.

**Tech Stack:** Kubernetes manifests, Argo CD `Application`, Kustomize

---

### Task 1: Add bootstrap structure

**Files:**
- Create: `bootstrap/root-application.yaml`
- Create: `bootstrap/root/kustomization.yaml`
- Create: `bootstrap/root/project-team9.yaml`

- [ ] Define the root application entrypoint for the `dev` cluster.
- [ ] Add a Kustomize bundle for child application resources.
- [ ] Add an AppProject that allows the expected repositories and namespaces.

### Task 2: Add child Applications and example repository credentials

**Files:**
- Create: `bootstrap/root/applications/external-secrets.yaml`
- Create: `bootstrap/root/applications/external-dns.yaml`
- Create: `bootstrap/root/applications/backend.yaml`
- Create: `bootstrap/root/applications/frontend.yaml`
- Create: `bootstrap/repositories/repository-team9-gitops.example-secret.yaml`
- Create: `bootstrap/repositories/repository-backend.example-secret.yaml`
- Create: `bootstrap/repositories/repository-frontend.example-secret.yaml`

- [ ] Add platform applications for `external-secrets` and `external-dns`.
- [ ] Add external repo-based applications for `backend` and `frontend`.
- [ ] Add example repository credential manifests with placeholder values.

### Task 3: Add cluster docs and verify manifests

**Files:**
- Create: `clusters/dev/bootstrap-info-configmap.yaml`
- Create: `clusters/dev/kustomization.yaml`
- Create: `clusters/dev/namespaces.yaml`
- Modify: `README.md`

- [ ] Add namespace manifests and a dev bootstrap checklist configmap.
- [ ] Document the required manual inputs and root apply flow.
- [ ] Run fresh manifest verification commands before closing the task.
