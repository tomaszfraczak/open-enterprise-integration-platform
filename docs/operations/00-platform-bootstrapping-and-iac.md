# Internal: Platform Bootstrapping and IaC Guidelines

## Document Purpose
**INTERNAL USE ONLY (also mirrored under public ops for platform engineers).**  
Guide for instantiating OEIP for a new client: repository topology, Day-0 / Day-1 order, and GitOps handoff.

---

## 1. Infrastructure repository topology

Blast-radius isolation uses **thin tenants** + one private platform core — not a fleet of overlapping IaC repos.

| Repository | Role |
|------------|------|
| **`platform-iac-core`** | Private Terraform modules (Azure lite ACA + full AKS), bootstrap (`client-prep`), Camel Quarkus template, lite-demo |
| **`platform-client-<client>-infra`** | Thin tenant: `platform.yaml`, backend config, lite/` and optional full `main.tf` |
| **`platform-gitops-core`** | Full profile only — Argo CD / Helm for Tier A in-cluster (APISIX, Keycloak, obs, ESO, …) |
| **`platform-docs`** | Public constitution (architecture, delivery, governance) |

Retired names (do not use): `platform-iac-foundation`, `platform-iac-tier-b` — folded into `platform-iac-core`.

---

## 2. Choose a profile first

| Profile | Day-0 | Day-1 | Cost posture |
|---------|-------|-------|----------------|
| **lite** | TF modules under `modules/azure/lite/*` via tenant `lite/` | Optional: Deploy Lite / Update Lite Apps | Lab / cost-capped |
| **full** | TF foundation + AKS + managed Tier B in `platform-iac-core` | Argo CD sync of `platform-gitops-core` | Funded enterprise |

Confirm strings: lite apply → type **`LITE`**; full apply → type **`APPLY`**.

Lab path: `platform-iac-core/docs/internal/operations/05-lab-cost-safe-path.md`.

---

## 3. Bootstrapping sequence

### Phase 0: Cloud preparation (both profiles)
* Run `platform-iac-core/bootstrap/client-prep.sh` (or `.ps1`) for the client/env.
* Creates mgmt + platform RGs (CAF `rg-oeip-…`), tfstate storage, deploy UAMI, GitHub Environment secrets.
* Do **not** use legacy NeutrOS / subscription-wide bootstrap scripts (`bootstrap/legacy/`).

### Phase 1a: Lite Day-0
* Tool: Terraform in tenant `lite/` (remote state key `lite.tfstate`).
* Workflow: **Deploy Lite Platform**.
* Actions: CAE, ACR Basic, Log Analytics, Redpanda/Postgres ACA, optional apps / edge / Keycloak / OTel.

### Phase 1b: Full Day-0
* Tool: Terraform in tenant root `main.tf` (separate state from lite).
* Actions: networking, AKS, ACR/Harbor decision, managed Postgres/Redis/Event Hubs (or Kafka), Key Vault, workload identity.

### Phase 2: Full Day-1 — GitOps handoff
* Install Argo CD on AKS; point at `platform-gitops-core` path from `platform.yaml` (`artifacts.gitopsPath`).
* From this moment, avoid `kubectl apply` for platform layers — Git is the control plane.
* Overlay must replace placeholders (`generic-tenant`, APISIX admin secrets via ESO).

### Phase 3: Control plane layers (full, via Argo)
Typical sync order: Security (ESO) → Observability → Identity → Edge (APISIX). Optional: service mesh.

### Phase 4: Golden path for domain teams
* Template: `platform-iac-core/templates/quarkus-camel-app` (publish as a GitHub template repo when ready).
* Reference flow: `platform-iac-core/examples/lite-demo`.
* Naming: `platform-[domain]-[service]-src` + companion gitops repo when on full.

---

## 4. "Secret Zero" management

* **Terraform state:** encrypted remote backend (Azure Storage + RBAC + versioning).
* **Cluster access (full):** federated OIDC to AKS — no long-lived admin kubeconfigs.
* **Lite secrets:** `capabilities.secrets.product` = `terraform` (state) or `azure-keyvault`; see tenant lite README.
* **Vault unseal (if used):** cloud KMS auto-unseal — never manual shard ops in production.
