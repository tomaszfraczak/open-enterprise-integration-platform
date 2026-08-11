# Golden Path and GitOps Delivery Workflow

## Document Purpose
This document outlines the standard delivery lifecycle for integration developers using the Platform. It explains how domain teams build, test, and deploy integrations without relying on a proprietary, closed-source UI portal.

---

## 1. Git as the Single Pane of Glass

### Philosophy
The platform adopts a **"Lean Platform Engineering"** approach. We do not force developers into a custom-built "Integration Portal" UI. Instead, the version control system (e.g., GitHub/GitLab) serves as the primary developer interface.

### Rationale
Developers are already comfortable with Git, Pull Requests, and IDEs. Building a custom web portal to click "Deploy" adds unnecessary abstraction. By using GitOps, every change—whether it is a line of Java code, a memory limit increase, or a new API route—is tracked, reviewable, and instantly reproducible.

---

## 2. The Delivery Workflow (Step-by-Step)

When a domain team needs to build a new integration (Tier C), they follow this standardized workflow. This workflow respects the **Responsibility Model**, where the Domain Team focuses solely on delivering code for the *Integration* and *Processing* layers, while the Platform Team manages the underlying CI/CD engine (Automation Layer) and repositories.

### Step 1: Bootstrap via Template (The Golden Path)
Instead of starting from an empty repository, copy or generate from the official platform template:

**Source of truth:** `platform-iac-core/templates/quarkus-camel-app`  
(Reference integration: `platform-iac-core/examples/lite-demo`.)

* **What it does:** Maven/Quarkus/Camel layout, Dockerfile, health, Helm chart (full), archetype toggles. The developer starts with thin routes + beans.
* **Naming:** `platform-[domain]-[service-type]-src` for source; `platform-[domain]-gitops` for manifests on the **full** profile.
* **When to use this template:** Tier C integrations (edge, bus, legacy connectors, EIP). For plain domain CRUD without an integration story, see [Integration Patterns §7](./02-integration-patterns-and-thin-routes.md) (Camel Quarkus vs plain Quarkus).
* **Profiles:** **lite** deploys images to Azure Container Apps; **full** pushes to the platform registry and syncs via Argo CD. See [Developer Onboarding (Azure)](./05-developer-onboarding-guide-azure.md).

### Step 2: Local Development and Testing
The developer writes their Apache Camel routes using their preferred IDE (IntelliJ, VS Code).
* **Testing:** Utilizing Camel's native testing frameworks and Quarkus Testcontainers, the integration is verified locally without needing access to the production platform.

### Step 3: Pull Request and CI Validation
Once the logic is ready, the developer opens a Pull Request (PR).
* **Automated Checks:** GitHub Actions (or GitLab CI) automatically run unit tests, enforce code linting, scan for vulnerabilities (e.g., exposed secrets), and validate that the architectural contracts (e.g., OpenAPI specs) are unbroken.
* **Approval:** Senior engineers or domain architects approve the PR.

### Step 4: Merge and deploy
Once merged to `main`, CI builds the container image (tag = Git SHA).

* **lite:** image → **ACR** → tenant Update Lite Apps / Deploy Lite.  
* **full:** image → platform registry → GitOps manifest bump → **Argo CD** syncs AKS. The developer does *not* run `kubectl apply` for platform-managed paths.  
* **Registry:** ACR is the Phase-1 default; Harbor is an optional later swap.

### Step 5: Operations via native tools
* **lite:** `bootstrap/lite-observe.sh`, ACA Log stream, Log Analytics (lenses A/B/C).
* **full:** Argo CD UI, Grafana/Loki (or swapped obs stack), API gateway metrics on the edge layer.