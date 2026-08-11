# Developer Onboarding Guide (Azure)

## Welcome
Build integrations on the OEIP **Golden Path**: thin Camel routes + Quarkus beans, contracts first, platform-owned edge / bus / identity.

---

## 1. Core philosophy: thin routes

* **Apache Camel (Camel Quarkus)** — routing, mediation, EIP, retries/DLQ (**Integration Layer**).
* **Quarkus CDI beans** — domain rules and non-trivial mapping (**Processing Layer**).
* Platform supplies edge, messaging, data, and observability as managed capabilities.

Patterns + error handling: [Integration Patterns](./02-integration-patterns-and-thin-routes.md) (§2.4 EH, §7 Camel vs plain Quarkus).

---

## 2. Two Azure profiles (pick one)

| | **lite** | **full** |
|---|----------|----------|
| Compute | Azure Container Apps | AKS |
| Messaging | Redpanda (ACA) | Event Hubs / Kafka |
| Edge | Optional APISIX ACA (standalone YAML) | APISIX Helm + etcd (+ Dashboard) |
| Delivery | GHA → ACR → ACA | CI → registry → Argo CD |
| When | Labs / cost-capped subs | Funded enterprise |

Tenant config lives in `platform-client-<client>-infra` (`platform.yaml` + TF). Platform modules live in private `platform-iac-core`.

---

## 3. Lifecycle of an integration

### Step 1: Bootstrap (Golden Path)
Do not start from an empty repo. Copy or generate from:

**`platform-iac-core/templates/quarkus-camel-app`**

(Published GitHub template name may vary; path above is the source of truth.)

End-to-end reference (REST → Avro Kafka → gRPC → Postgres + DLQ):  
`platform-iac-core/examples/lite-demo`

Naming: `platform-[domain]-[service]-src` (and `platform-[domain]-gitops` on full).

### Step 2: Local development
* `quarkus dev` / Camel tests / Compose under the template or lite-demo.
* You do not need Azure to prove the route locally.

### Step 3: Deploy
* **lite:** build images → ACR → **Update Lite Apps** / Deploy Lite (tenant workflows).
* **full:** CI pushes image → GitOps repo bump → **Argo CD** syncs AKS (no `kubectl apply` for platform paths).

Registry: **ACR** in Phase 1; Harbor remains an optional swap later — do not assume Harbor on day one.

---

## 4. Pre-deployment checklist (Definition of Done)

* [ ] Stateless pods; durable state only in bus / DB  
* [ ] Health: `/q/health/live` and `/q/health/ready`  
* [ ] `Correlation-ID` + structured `service=` / `event=` logs  
* [ ] Error handling: `onException` + DLQ / HTTP 4xx·503 (see §2.4)  
* [ ] Contracts: OpenAPI / Avro / proto committed  
* [ ] Resources: CPU/memory limits on the target profile (ACA or Helm)
