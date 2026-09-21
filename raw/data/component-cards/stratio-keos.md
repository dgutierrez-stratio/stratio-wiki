---
id: stratio-keos
name: "Stratio KEOS"
aliases: ["Kubernetes-based Enterprise Operating System", "keos-installer"]
category: unknown
repo: not-migrated
scope: unknown
lifecycle: active
last_reviewed: 2026-09-21
---

# Stratio KEOS

> The Kubernetes-based platform foundation everything else in the fleet runs on — not a component deployed on its own.

**Not a discrete GitOps deployable — this is the platform foundation's name.** Stratio KEOS
("Kubernetes-based Enterprise Operating System") is the substrate that `keos-apps`,
`keos-system-services`, and `keos-fleet` are literally named after and run on top of — it's
installed before GitOps/Flux even exists on a cluster, via a separate tool called the
keos-installer (handed off to by Stratio Cloud Provisioner after it bootstraps the raw cluster).
No component literally named `keos` exists in either `keos-apps` or `keos-system-services`. See
each system service's own card (Stratio GoSec, the four datastore operators, etc.) for the
pieces KEOS's installer actually deploys — the Dependencies, Used by, Ingress, and Security
sections below don't meaningfully apply to KEOS as a bounded component.

## 1. Overview

Stratio KEOS is Stratio's Kubernetes-based platform foundation: it takes raw Kubernetes and adds
everything needed to run it in production — multitenancy, security (Vault, Kyverno, LDAP/
Kerberos), data security (GoSec), HTTP exposure, SSO, storage, backup/restore, monitoring,
logging, and Stratio application lifecycle management (Command Center). It's installed via a
separate tool, the keos-installer (built on Kubespray/Ansible), in phases: precheck, prepare
(on-prem only), K8s-install (on-prem only — installs vanilla Kubernetes plus CoreDNS, Calico, and
Flux), and system-services (deploys the platform capabilities listed above). Everything installed
in this phase runs under a dedicated `keos` tenant, in namespaces prefixed `keos-*` (e.g.
`keos-core`, `keos-auth`, `keos-ingress`) — which is why this fleet's own GitOps repos
(`keos-apps`, `keos-system-services`, `keos-fleet`) carry the same name.

## 2. Classification

| | |
|---|---|
| **Type** | unknown — not a Kubernetes workload; the platform foundation's name |
| **Deployment scope** | Not applicable — it has no discrete GitOps identity of its own |
| **Home repo** | Not applicable |

## 3. Dependencies

Not applicable in the normal sense — KEOS is the foundation other components depend on, not a
consumer of them. Its one real upstream relationship: Stratio Cloud Provisioner bootstraps the
raw cluster and infrastructure, then hands off to the keos-installer to complete setup.

**Used by:**
- Not applicable — every component cataloged in this wiki runs on top of KEOS; it isn't "used
  by" any one of them specifically.

## 4. Flavors & Deployment Options

| Option | Choices | What changes |
|---|---|---|
| Installer resource-allocation flavour | production (default) \| development \| custom | Replica counts and resource sizing for all system-services deployments — development is documented as testing-only, never for production. |
| Storage provider | Ceph (not recommended for production) \| csi-aws \| local-path \| nfs \| custom | Which storage backend the cluster's system-services use. |
| Backup target (via Velero) | AWS \| Azure \| GCP \| on-prem MinIO | Where cluster-level backups are stored. |

## 5. Ingress / Exposed Endpoints

Not applicable directly — KEOS's install process deploys `ingress-nginx` as one of its system
services, which every other component then uses. That's infrastructure it provisions, not an
endpoint of "KEOS" itself.

## 6. Security & Secrets

| | |
|---|---|
| **Authentication** | Not applicable as a bounded component |
| **Credentials it holds** | Not applicable |
| **Notable access controls** | Not applicable — KEOS's "Security" documentation describes the platform-wide authentication/authorization architecture it establishes (API server access controls, admin UI SSO via ingress-nginx + oauth2-proxy + Stratio SIS, RBAC), not a security surface belonging to a discrete KEOS service |

---
_For exact file locations, dependency wiring, and deployment-mode configuration mechanics, see the
`x-ray` skill. Internal architecture — tech stack, internal data/control flow, scalability/HA/DR,
API contracts — is intentionally not covered here; it belongs to a separate architecture card._
