---
id: stratio-data-marketplace
name: "Stratio Data Marketplace"
aliases: ["datamarket-agent", "datamarket-api", "datamarket-ui"]
category: business-application
repo: keos-apps
scope: cluster-wide
lifecycle: active
last_reviewed: 2026-09-21
---

# Stratio Data Marketplace

> A marketplace where producers publish data products and consumers request access to them — runs as a plugin inside Stratio Data Governance.

## 1. Overview

Stratio Data Marketplace lets data producers create and publish data products, and lets consumers
browse and request access to them, managing the required cross-platform permissions
automatically. Some data products can optionally be exposed in a public space, letting anonymous
users browse — and in some cases directly consume — public data assets, including DCAT-AP-ES
catalog support. It functions as a plugin within Stratio Data Governance rather than as a
standalone component, and always requires a running Data Governance instance. A tenant-scoped
`datamarket-agent` handles the permission-sync side, propagating consumer/producer access grants
out to Discovery, Virtualizer, Rocket, and (optionally) GenAI.

## 2. Classification

| | |
|---|---|
| **Type** | Business application |
| **Deployment scope** | Cluster-wide API/UI (`datamarket-api`/`datamarket-ui`, deployed as controllers inside the Data Governance core service) plus a tenant-scoped permission-sync agent (`datamarket-agent`, one per tenant that enables it) |
| **Home repo** | keos-apps |

## 3. Dependencies

```
Stratio Data Marketplace (datamarket-api / datamarket-ui, inside governance)
└─ Stratio Data Governance                          (required — hosts it as a plugin)
     └─ Stratio PostgreSQL / Search Engine            (governance's own dependencies)

Stratio Data Marketplace — datamarket-agent (per tenant)
├─ Stratio Discovery                                 (required)
├─ Stratio Virtualizer                                (typically wired)
├─ Stratio Rocket                                     (typically wired)
└─ Stratio GenAI                                      (optional — seen commented out in one tenant)
```

**Used by:**
- None found. No other component lists Stratio Data Marketplace as its own dependency — it's a
  leaf consumer of Discovery, Virtualizer, Rocket, and Governance.

## 4. Flavors & Deployment Options

| Option | Choices | What changes |
|---|---|---|
| Enablement | on \| off (single toggle for both API and UI together) | Whether the marketplace plugin is deployed alongside the Data Governance core. |
| Public data space | on \| off | Whether some data products are exposed to anonymous, unauthenticated users. |
| DCAT-AP-ES protocol support | on \| off | Whether the public catalog supports DCAT-AP-ES import/export. |
| Size | S \| M \| L | Resource sizing tier for the datamarket-api/ui containers. |

## 5. Ingress / Exposed Endpoints

| Where | Protected by | What it's for |
|---|---|---|
| Marketplace API and UI (admin/internal path, under governance's ingress) | SSO (oauth2-proxy), optional mTLS | Browsing and managing data products, contracts, and permissions as an authenticated user. |
| Public data space (observed in a legacy example as a separate public ingress) | None — intentionally anonymous, hardened with ModSecurity/OWASP rule sets | Letting anonymous users browse and, for open products, directly consume public data assets. |

## 6. Security & Secrets

| | |
|---|---|
| **Authentication** | SSO via oauth2-proxy for the main marketplace UI/API; the public/open data space is intentionally unauthenticated, hardened instead with ModSecurity/OWASP rule sets |
| **Credentials it holds** | A generated Datamarket API key, synced as a Kubernetes secret and used for its Discovery integration; GoSec-issued impersonator ACLs letting its consumer group act through Virtualizer and manage the Rocket catalog |
| **Notable access controls** | A post-install job provisions per-tenant GoSec groups (consumer and producer user groups) and maps them to built-in Consumer/Producer roles in Data Governance, plus a consumer role in GenAI |

---
_For exact file locations, dependency wiring, and deployment-mode configuration mechanics, see the
`x-ray` skill. Internal architecture — tech stack, internal data/control flow, scalability/HA/DR,
API contracts — is intentionally not covered here; it belongs to a separate architecture card._
