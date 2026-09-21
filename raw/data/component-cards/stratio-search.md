---
id: stratio-search
name: "Stratio Search"
aliases: ["search-engine"]
category: system-service
repo: keos-apps
scope: tenant
lifecycle: active
last_reviewed: 2026-09-21
---

# Stratio Search

> A search-engine layer built on top of OpenSearch, adding domains, indexing workflows, and product integration — primarily used by Data Governance to search its metadata catalog and business glossary.

## 1. Overview

Stratio Search is a search engine built on the OpenSearch API, layered on top of a Stratio
OpenSearch Operator cluster (the underlying datastore, documented on its own card) rather than
replacing it. It adds product-level capabilities OpenSearch doesn't provide on its own: multiple
independent search "domains" per installation, hierarchical categories, compound fields,
post-filtering, nested documents, autocomplete/spell-check, and integration with the platform's
security (GoSec) and Command Center. It runs as three microservices — Manager (domain
configuration), Indexer (document ingestion), and Searcher (querying) — while still allowing
native OpenSearch queries directly when needed. Its primary consumer in this fleet is Stratio
Data Governance, which uses it to search discovered metadata and the business glossary.

## 2. Classification

| | |
|---|---|
| **Type** | System service (tenant-scoped internal glue over OpenSearch) |
| **Deployment scope** | Tenant — one instance per tenant that enables it |
| **Home repo** | keos-apps |

## 3. Dependencies

```
Stratio Search (search-engine)
└─ Stratio OpenSearch                (required — the underlying datastore it's layered on top of)
```

**Used by:**
- Stratio Data Governance — an optional dependency; when enabled, it also gates whether
  Governance's scheduler agent (which drives indexing jobs) runs at all.

## 4. Flavors & Deployment Options

| Option | Choices | What changes |
|---|---|---|
| Size | S \| M \| L | Scaffolded in GitOps but not currently differentiated — all three tiers are identical in this fleet. |
| Microservices | Manager, Indexer, Searcher (all on by default) | Each is independently toggleable at the chart level, though a typical instance runs all three. |
| Indexation mode | Total (batch, zero-downtime swap) \| Partial (near-real-time create/update) \| Delta (near-real-time partial updates) | How documents get indexed into a domain. |

## 5. Ingress / Exposed Endpoints

| Where | Protected by | What it's for |
|---|---|---|
| Manager, Indexer, and Searcher APIs, each under its own path on the admin domain | SSO via oauth2-proxy, optional mutual TLS | Configuring search domains (Manager), ingesting documents (Indexer), and querying (Searcher). |

## 6. Security & Secrets

| | |
|---|---|
| **Authentication** | SSO via the platform's oauth2-proxy on all three exposed APIs, plus JWT validation using the shared ingress-nginx signing keys |
| **Credentials it holds** | A per-microservice mTLS certificate, and generated keystore/truststore passwords and an OAuth client secret (used to self-register as an SSO service) |
| **Notable access controls** | Each microservice is granted broad GoSec ACLs (read/write/delete/manage on all indices, manage/monitor on the cluster) under its own GoSec identity; a tenant-isolation setting restricts mutual-TLS client access to a specific whitelisted caller (Data Governance's scheduler agent) |

---
_For exact file locations, dependency wiring, and deployment-mode configuration mechanics, see the
`x-ray` skill. Internal architecture — tech stack, internal data/control flow, scalability/HA/DR,
API contracts — is intentionally not covered here; it belongs to a separate architecture card._
