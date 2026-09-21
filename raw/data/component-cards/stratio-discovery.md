---
id: stratio-discovery
name: "Stratio Discovery"
aliases: []
category: business-application
repo: keos-apps
scope: tenant
lifecycle: active
last_reviewed: 2026-09-21
---

# Stratio Discovery

> A Metabase-based BI/dashboarding tool that lets non-technical users explore and visualize platform data without writing SQL.

## 1. Overview

Stratio Discovery is a data-discovery and dashboarding tool, built on Metabase, that lets users
explore company data and build reports and dashboards without needing SQL or other technical
skills. It connects natively to Stratio Virtualizer and Stratio PostgreSQL via custom drivers,
plus a range of external sources (Oracle, MySQL, PostgreSQL, MongoDB, Google Analytics, BigQuery,
Amazon Redshift). It's aimed at business users who need to ask questions of governed data and
share the answers as dashboards, while access to virtualized data is enforced under each user's
own identity so row- and column-level security is preserved.

## 2. Classification

| | |
|---|---|
| **Type** | Business application |
| **Deployment scope** | Tenant — one instance per tenant that enables it |
| **Home repo** | keos-apps |

## 3. Dependencies

```
Stratio Discovery
├─ Stratio PostgreSQL (pooled through PgBouncer)   (required — Discovery's own metadata/app database)
└─ Stratio Virtualizer                              (required in most observed tenants — registered
                                                       as a "Data Collections" source post-install)
```

**Used by:**
- Stratio GenAI — depends on Discovery (togglable)
- Stratio Data Marketplace's `datamarket-agent` — depends on Discovery

## 4. Flavors & Deployment Options

| Option | Choices | What changes |
|---|---|---|
| Size | S \| M \| L | Resource sizing tier. |
| Extra database drivers | Extra JARs added to the Metabase plugins folder (e.g. Oracle), configured via Stratio Command Center | Which additional external databases Discovery can connect to. |
| Enablement | on \| off (per tenant, and as an `enabled` flag on other components' dependency on it) | Whether Discovery is deployed for a given tenant. |

## 5. Ingress / Exposed Endpoints

| Where | Protected by | What it's for |
|---|---|---|
| Discovery web UI (`https://discovery.<domain>/discovery`) | SSO via the platform's oauth2proxy (sets a `stratio-cookie` JWT); the underlying Metabase session additionally uses its own session cookie | Browsing/asking questions of governed data and building and sharing dashboards. |

## 6. Security & Secrets

| | |
|---|---|
| **Authentication** | SSO via the platform's oauth2proxy, layered with Metabase's own session cookie for its internal API |
| **Credentials it holds** | A Vault-issued TLS certificate and secrets bundle provisioned per release; admin credentials are used transiently during post-install setup to obtain an admin JWT, not stored long-term by Discovery itself |
| **Notable access controls** | Discovery is granted an "impersonator" GoSec ACL against Virtualizer, so it queries virtualized data under each end user's own identity rather than a shared service identity — preserving row- and column-level security |

---
_For exact file locations, dependency wiring, and deployment-mode configuration mechanics, see the
`x-ray` skill. Internal architecture — tech stack, internal data/control flow, scalability/HA/DR,
API contracts — is intentionally not covered here; it belongs to a separate architecture card._
