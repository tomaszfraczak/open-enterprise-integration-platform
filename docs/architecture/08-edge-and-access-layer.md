# Edge and Access Layer Standards

## Document Purpose
This document defines the standards for the entry point of the Platform. The Edge Layer (Tier B) is responsible for traffic routing, edge security, and protocol mediation.

---

## 1. Edge Gateway Architecture (Tier B)

The Edge Layer acts as the "Front Door," centralizing cross-cutting concerns like SSL termination, rate limiting, and request transformation.

---

## 2. Multi-Cloud Strategy and Component Swap

This layer is interchangeable to fit the target environment's network topology:

| OSS Component | Azure Equivalent | AWS Equivalent | GCP Equivalent |
| :--- | :--- | :--- | :--- |
| **Apache APISIX / Kong** | Azure API Management (APIM) | AWS API Gateway | Google Cloud API Gateway |
| **Ingress NGINX** | Azure Application Gateway | AWS ALB / NLB | GCP HTTP(S) Load Balancer |
| **WAF** | Azure WAF | AWS WAF | Google Cloud Armor |

---

## 3. Security Policies
* **Zero-Trust Ingress:** No internal service may expose a public LoadBalancer; all traffic must pass through the Edge Layer.
* **Rate Limiting:** Mandatory burst and average limits per consumer.
* **Operator console (APISIX Dashboard):** Full profile exposes the embedded Admin UI (`/ui/` on the Admin port) behind allow-lists / SSO — see internal runbook `apisix-dashboard.md` and [upstream Dashboard docs](https://apisix.apache.org/docs/apisix/dashboard/). Lite profile uses file-driven standalone APISIX without Dashboard.
