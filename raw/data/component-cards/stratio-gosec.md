---
id: stratio-gosec
name: "Stratio GoSec"
aliases: ["gosec-agent", "gosec-operator"]
category: system-service
repo: keos-system-services
scope: cluster-wide
lifecycle: active
last_reviewed: 2026-09-21
---

# Stratio GoSec

> The platform's centralized identity, authorization, and auditing backbone — the IAM system nearly every other component registers into.

## 1. Overview

Stratio GoSec is the integrated security solution for the platform: centralized identity
management (local, LDAP, or Active Directory), real-time authorization policy enforcement,
row/column-level data filtering, attribute/tag-based policies fed from Stratio Data Governance,
and centralized auditing. Authentication itself is delegated to the Stratio Identity Server
(SIS), which provides single sign-on across the platform's modules via OAuth2/OIDC/CAS, LDAP, or
SAML federation, with optional MFA. A cluster-wide core service (with its own admin console and
API) is complemented by tenant-scoped `gosec-agent` instances that register individual datastores
(PostgreSQL, OpenSearch) into GoSec's authorization model. Nearly every other business application
in the platform — Rocket, Data Governance, Discovery, GenAI, Data Marketplace — registers its own
service identities and access policies against GoSec during install.

## 2. Classification

| | |
|---|---|
| **Type** | System service |
| **Deployment scope** | Cluster-wide core service (`gosec` + `gosec-operator`, one instance per cluster) with tenant-scoped, per-datastore agents (`gosec-agent`, deployed as a Postgres or OpenSearch flavor per tenant) |
| **Home repo** | keos-system-services (core service and operator); keos-apps (tenant-scoped agent) |

## 3. Dependencies

```
Stratio GoSec (core)
├─ Stratio PostgreSQL (pooled through PgBouncer)   (required — its own database)
├─ Vault                                            (required — secrets/KMS backend)
├─ Platform LDAP/IDP                                (required — identity backend)
└─ Stratio Data Governance                          (required — for security-attribute/tag lookups)

Stratio GoSec — gosec-agent (per tenant, per datastore)
└─ The datastore it's registering (Postgres or OpenSearch)   (required)
```

**Used by:**
- Nearly every business application in this fleet registers against GoSec via its own
  post-install job — confirmed for Stratio Rocket, Stratio Data Governance, Stratio Discovery,
  Stratio GenAI, and Stratio Data Marketplace's `datamarket-agent`.
- In tenant configs, `gosecAgentPostgres`/`gosecAgentOpensearch` are listed as explicit
  dependencies of Stratio GenAI, Stratio Intelligence, Stratio Virtualizer, and Stratio
  Connectors.

## 4. Flavors & Deployment Options

| Option | Choices | What changes |
|---|---|---|
| Identity backend | External IdP \| Internal IdP (platform-managed LDAP) | Where GoSec's identity management is sourced from. |
| Authentication protocol | OAuth2, OIDC, CAS, LDAP, SAML (external federation), optional MFA | How users authenticate through SIS. |
| Datastore agent type | Postgres \| OpenSearch | Which datastore a given gosec-agent instance registers into GoSec. |
| Size | S \| M \| L | Resource sizing tier, for both the core operator and per-tenant agents. |

## 5. Ingress / Exposed Endpoints

| Where | Protected by | What it's for |
|---|---|---|
| GoSec admin console (`/gosec/ui`) and its REST API (`/gosec/baas`) | SSO via oauth2-proxy (browser login redirect) | Managing users, groups, roles, and access policies platform-wide. |

## 6. Security & Secrets

| | |
|---|---|
| **Authentication** | Delegated to SIS (Stratio Identity Server) for SSO across the platform, supporting OAuth2/OIDC/CAS/LDAP/SAML with optional MFA; internal calls between GoSec's own microservices use mutual TLS with certificate-CN-based allow-lists |
| **Credentials it holds** | Vault-backed TLS certificates and a JWT signing keypair (shared with the ingress controller); Vault itself is used as GoSec's KMS backend |
| **Notable access controls** | Other applications register their service identities and access policies into GoSec declaratively via Kubernetes CRDs (GosecPolicy, GosecBinding, GosecUser, GosecGroup, GosecRole), reconciled by the gosec-operator; GoSec's own internal microservices restrict sensitive operations (user/policy management) to an explicit allow-list of trusted service certificate names |

---
_For exact file locations, dependency wiring, and deployment-mode configuration mechanics, see the
`x-ray` skill. Internal architecture — tech stack, internal data/control flow, scalability/HA/DR,
API contracts — is intentionally not covered here; it belongs to a separate architecture card._
