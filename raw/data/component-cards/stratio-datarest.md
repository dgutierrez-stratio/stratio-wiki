---
id: stratio-datarest
name: "Stratio DataREST"
aliases: ["bdl-datarest", "dg-datarest"]
category: business-application
repo: keos-apps
scope: tenant
lifecycle: active
last_reviewed: 2026-09-21
---

# Stratio DataREST

> An auto-generated REST API over a tenant's data sources, modeled after PostgREST.

## 1. Overview

Stratio DataREST (deployed in GitOps as `bdl-datarest`) is a Stratio Generative AI Data Fabric
component that automatically generates a REST API from a tenant's data sources, in the style of
PostgREST. Once deployed, it connects to a configured backend — PostgreSQL, Oracle, SQL Server,
or Stratio Virtualizer/Crossdata — and exposes its schemas and tables as standard REST endpoints
(GET/POST/PUT/DELETE), browsable and testable through a built-in Swagger UI. It's aimed at
consumers who want direct HTTP access to tenant data without writing custom API code. Access
control is enforced through the platform's identity mechanisms, with JWT, mTLS, and header-based
authentication all supported depending on configuration.

In this fleet's sampled tenant configs, DataREST's enablement block was present but commented out
(inactive) in both examples found — treat it as available-but-not-yet-turned-on in those tenants
rather than actively serving traffic there.

## 2. Classification

| | |
|---|---|
| **Type** | Business application |
| **Deployment scope** | Tenant — one instance per tenant that enables it |
| **Home repo** | keos-apps |

## 3. Dependencies

```
Stratio DataREST
├─ Stratio PostgreSQL          (required — for the pginternal / pgmd5 / pgtls backend types)
└─ Stratio Virtualizer         (required — only when the backend type is "virtualizer")
```

Oracle and SQL Server backend types connect via direct connection details and have no modeled
GitOps dependency.

**Used by:**
- None found. It's a leaf service consumed by external API clients over HTTP, not by other
  internal Stratio components.

## 4. Flavors & Deployment Options

| Option | Choices | What changes |
|---|---|---|
| Backend type | PostgreSQL (internal TLS / MD5 / TLS-cert auth) \| Oracle 12c+ \| Oracle 11g \| Microsoft SQL Server \| Stratio Virtualizer (Crossdata) | Which data source DataREST connects to and exposes as a REST API. |
| Size | S \| M \| L | Resource sizing tier. |
| Swagger UI | on \| off (generally on) | Whether the interactive API explorer is available alongside the REST API. |
| Endpoint exposure | per-verb toggles (select/insert/update/delete/joins), admin endpoints | Which REST operations and admin endpoints are reachable. |
| Connection pooling (Postgres backend) | on \| off (default off) | Whether Postgres connections are pooled through PgBouncer. |

## 5. Ingress / Exposed Endpoints

| Where | Protected by | What it's for |
|---|---|---|
| REST API + Swagger UI, path-prefixed per tenant/release (e.g. `/service/<tenant-service>/<schema>/<table>`) | mTLS client-certificate verification at the ingress edge, plus a choice of JWT, TLS-client, or header-based (`X-Tenant-Id`/`X-User-Id`) authentication | Calling the auto-generated REST API and browsing it interactively via Swagger. |

## 6. Security & Secrets

| | |
|---|---|
| **Authentication** | JWT (validated against a platform-managed RSA public key), TLS client-certificate auth, and header-based auth — configurable, more than one can be enabled at once |
| **Credentials it holds** | A TLS certificate and generated database/keystore passwords per release; for the internal-Postgres backend, broad GoSec-managed Postgres ACLs (including a superuser role attribute); for the Virtualizer backend, Crossdata ACLs including impersonation rights |
| **Notable access controls** | A dedicated GoSec user/policy/binding is provisioned per release; audit logging of sensitive data is configurable and varies by backend-type |

---
_For exact file locations, dependency wiring, and deployment-mode configuration mechanics, see the
`x-ray` skill. Internal architecture — tech stack, internal data/control flow, scalability/HA/DR,
API contracts — is intentionally not covered here; it belongs to a separate architecture card._
