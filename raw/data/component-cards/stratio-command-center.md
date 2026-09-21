---
id: stratio-command-center
name: "Stratio Command Center"
aliases: ["cct"]
category: system-service
repo: not-migrated
scope: cluster-wide
lifecycle: active
last_reviewed: 2026-09-21
---

# Stratio Command Center

> The product's operations center — publishes application descriptors ("universes") and manages the install/upgrade/uninstall lifecycle of every other application in the platform.

**Not found via the standard GitOps component pattern** — Stratio Command Center isn't defined
under `keos-apps/components/` or `keos-system-services/components/` the way other components are.
It's still deployed through a legacy Helm chart-of-charts (`cct`, namespace `keos-cct`), and
GoSec's own configs contain an explicit `TODO: Once every cluster is migrated to gitops we can
remove cct CNs from these lists` — confirming migration to the native GitOps pattern is underway
but not complete. Unlike a typical not-yet-migrated component, real deployment facts *are*
available here: the details below come from a captured point-in-time deployment snapshot
(cluster `eosdev`, dated 2026-06-08) rather than a live templated GitOps source, so treat them as
fleet-real but not guaranteed current.

## 1. Overview

Stratio Command Center is the operations center for the Stratio product — it manages the
lifecycle of every other application in the platform, letting operators deploy, edit, update, and
uninstall applications through "universes": sets of JSON descriptors that define an application's
configuration, install/upgrade tasks, customizable parameters, and metadata. It also hosts the
"Stratio menu," a cross-product navigation panel that links out to the other components' own UIs
(Virtualizer, Rocket, Intelligence, Discovery, History Server, BDL UI, OpenSearch Dashboards,
GoSec). Rather than being a service other components depend on in the GitOps graph, Command
Center is the tool that installs and manages those components in the first place.

## 2. Classification

| | |
|---|---|
| **Type** | System service |
| **Deployment scope** | Cluster-wide — one instance per cluster, in its own dedicated `keos-cct` namespace (inferred from the namespace pattern and the captured snapshot, not confirmed via the standard cluster-wide `resourcesetinputproviders.yaml` mechanism) |
| **Home repo** | Not yet migrated to the standard GitOps component pattern — deployed via a legacy Helm chart-of-charts |

## 3. Dependencies

```
Stratio Command Center (6 sub-services: orchestrator, central-configuration,
                         paas-services, universe, ui, applications-query)
├─ Stratio GoSec (gosec-identities-daas / gosec-services-daas)   (required)
├─ SIS / platform SSO (sis-api)                                  (required)
├─ Vault                                                         (required — TLS keystores, CA trust)
├─ LDAP / Kerberos (platform IdP)                                (required)
└─ Shared core PostgreSQL (via its own PgBouncer instance)       (required)
```

Sourced from a captured deployment snapshot, not a live GitOps template — representative, but not
guaranteed current.

**Used by:**
- None found as a formal GitOps dependency edge. The relationship runs the other way — Command
  Center installs and manages other applications through universe descriptors, rather than being
  depended on by them. GoSec explicitly trusts Command Center's service accounts (whitelisted in
  its ACL/certificate configs), confirming Command Center calls into GoSec, not the reverse.

## 4. Flavors & Deployment Options

| Option | Choices | What changes |
|---|---|---|
| Monitoring integration | on \| off (default off in this fleet) | Whether Prometheus metrics collection for Command Center is enabled. |

No size-tier (S/M/L) options were found, unlike most other components — it appears to run as a
fixed single-replica deployment per sub-service in the snapshot examined.

## 5. Ingress / Exposed Endpoints

| Where | Protected by | What it's for |
|---|---|---|
| Command Center UI (`/cct/ui`) and its 5 backing API services (applications-query, central-configuration, orchestrator, paas-services, universe), all under the platform's admin hostname | SSO (oauth2-proxy, with browser login redirect) required; optional mTLS client-certificate verification | The main operations console for deploying, editing, updating, and uninstalling applications platform-wide |

## 6. Security & Secrets

| | |
|---|---|
| **Authentication** | SSO via oauth2-proxy, with optional TLS client-certificate verification at the ingress edge; JWT keys sourced from the shared ingress-nginx JWT secret bundle |
| **Credentials it holds** | A Vault-issued TLS keystore and CA trust bundle per sub-service; Postgres credentials for its own database schema; (per an example/reference manifest, not confirmed live) a Vault role granting broad read/list access across Vault |
| **Notable access controls** | Role-group mapping ties `cct-cluster-admin` → cluster-admin, `cct-tenant-admin` → tenant-admin, `cct-guest` → tenant-user; GoSec explicitly whitelists Command Center's service identities for user/group/tenant/profile ACL operations, marked in GoSec's own config as a legacy trust relationship slated for removal once GitOps migration completes |

---
_For exact file locations, dependency wiring, and deployment-mode configuration mechanics, see the
`x-ray` skill. Internal architecture — tech stack, internal data/control flow, scalability/HA/DR,
API contracts — is intentionally not covered here; it belongs to a separate architecture card._
