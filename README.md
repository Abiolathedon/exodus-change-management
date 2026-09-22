# EXODUS — Enterprise Infrastructure Automation & Platform Engineering Lab

> **What is this?** A self-directed lab where I build and operate a fleet of
> enterprise Linux + Windows servers the way a real **platform / SRE team** would —
> automating everything as code, hardening and securing it, integrating it with
> **Active Directory** for centralised identity, monitoring it end-to-end, and
> **governing every change with a real ITIL-style change-management process.**
> It is **infrastructure / platform** work: the automated, observable, governed
> *foundation* that applications run on — and my hands-on proof of infrastructure-
> and platform-engineering skill.
>
> **Actively developed.** This README reflects the current state; the roadmap shows
> what's done and what's next.

---

## Why this stands out

Most infrastructure projects show automation. **EXODUS also shows how a real
engineering organisation *secures, identifies, operates, and governs* that
infrastructure:**

- 🔁 **Config-as-code, fleet-wide** — reusable Ansible roles, idempotent, version-pinned.
- 🔐 **Secrets done right** — Ansible Vault; **no plaintext credentials anywhere in the repo.**
- 🪪 **Centralised identity** — RHEL fleet **joined to Windows Active Directory**: Kerberos-backed auth, SSSD, realm membership, and **AllowGroups RBAC** enforcement.
- 🛡️ **Security hardening** — IAM least-privilege cleanup and **SSH hardening** applied and re-standardised fleet-wide.
- 📊 **End-to-end observability** — metrics (Prometheus/Grafana) *and* centralised logging (Vector → OpenSearch).
- 🧾 **Real change governance** — an **ITIL 4-aligned** change-classification policy (Standard / Normal / Emergency tiers), Change Requests, tiered templates, closure records, and Git/issue traceability.
- 🚨 **Incident response** — real incidents diagnosed and documented as **operational runbooks** (detection → root cause → fix → lessons).

The security, identity, and governance depth is what separates this from a typical
home lab. **This is run like a workplace, not a sandbox.**

---

## In one paragraph

A fleet of **RHEL 9.2** servers plus a **Windows Server Active Directory** domain,
automated with **Ansible** (config-as-code), secured with **Ansible Vault**,
identity-integrated with **AD** (Kerberos, SSSD, RBAC), hardened (IAM least-privilege,
SSH), observed end-to-end (**Prometheus + Grafana** for metrics; **Vector → OpenSearch**
for logs), fronted by **NGINX** — all operated through a real **change-management
workflow** (change requests, tickets, runbooks, Git traceability). Along the way I've
rolled out AD across the fleet, hardened access, handled real incidents, and performed
credential rotations. Not tutorials — a running system, built and operated to
enterprise standards.

---

## What this demonstrates (and the roles it maps to)

Built to evidence the skills these roles require — **Platform Engineer · Site
Reliability Engineer (SRE) · DevOps Engineer · Linux / Infrastructure Engineer ·
Automation Engineer.**

| Capability | Demonstrated by |
|---|---|
| Infrastructure / Config as Code | Reusable, parameterised Ansible roles bringing nodes to a defined desired state, idempotently |
| Secrets management | Ansible Vault — credentials encrypted at rest, injected at deploy time; **no plaintext secrets in the repo** |
| Identity & access (cross-platform) | RHEL fleet **joined to Windows Active Directory** — Kerberos, SSSD, realm membership, AD-integrated DNS |
| RBAC & access control | Centralised **AllowGroups RBAC** enforcement, validated fleet-wide (CR-012) |
| Security hardening | **IAM least-privilege** cleanup (CR-011) and **SSH hardening** re-standardised across the fleet (CR-012) |
| Centralised logging | `vector` role deploys the logging agent fleet-wide (config from template, password from Vault) → OpenSearch |
| Monitoring & alerting | Prometheus + node_exporter → Grafana dashboards; Alertmanager |
| Automated patching | Tiered `patching` role + pipeline: test → staging → production, rolling (serial), with security-only filtering, reboot detection, and error handling |
| Change governance (ITIL 4) | Change Requests (CR-011 → CR-013), classification policy (Standard/Normal/Emergency), tiered GitHub issue templates, closure records |
| Incident response | Operational runbooks — e.g. the services-01 disk-fill incident: detection → root cause → fix → lessons |
| Linux administration | RHEL 9.2, systemd, package lifecycle, network (NetworkManager/nmcli) |
| GitOps / version control | Conventional commits, CR traceability, archived (never deleted) prior configs |

---

## Security, identity & governance (the differentiators)

Beyond automation, EXODUS is run with the controls a real organisation expects:

**Identity & access (Active Directory)**
- The RHEL fleet is **joined to a Windows AD domain** — Kerberos-backed authentication,
  SSSD, realm membership, and AD-integrated DNS.
- **Centralised RBAC** via `AllowGroups` — access governed by AD group membership,
  validated across the fleet.

**Hardening**
- **IAM least-privilege** review and cleanup (CR-011).
- **SSH hardening** applied and re-standardised fleet-wide, with deterministic parity
  validation (CR-012).

**Change management (ITIL 4-aligned)**
- Every significant change is a **Change Request** (CR-011 → CR-013), classified by a
  documented **policy** into **Standard / Normal / Emergency** tiers so governance is
  matched to risk.
- **Tiered templates** (lightweight Standard vs fuller Enterprise), GitHub issue
  tracking, and **closure records** with evidence.
- **Traceability** — commits reference their CR/issue; prior configs are archived,
  never deleted; changes follow rollback-first, validate-before-reload methodology.

**Incident response**
- Incidents are written up as reusable **runbooks**. Example: a disk-fill incident on
  `services-01` (an 18 GB error-log cascade) — documented end to end, with the
  diagnostic sequence, the source-first fix, and the lessons (a missing disk-usage
  alert, now on the backlog).

`docs/` holds the classification policy, the CR records (CR-011 → CR-013), and the runbooks.

---

## The fleet

Eight nodes on an isolated network (7× RHEL 9.2 + 1× Windows AD), grouped both
**functionally** and by **environment tier** (test → staging → production):

| Node | Role |
|------|------|
| `exodus-automation-01` | Ansible **control node** |
| `exodus-platform-01`   | Platform / test tier |
| `exodus-services-01`   | Services (production tier); IAM/RBAC pilot node |
| `exodus-observability-01` | Prometheus + Alertmanager (staging tier) |
| `exodus-grafana-01`    | Grafana |
| `exodus-ingress-01`    | NGINX ingress |
| `exodus-logging-01`    | OpenSearch (log store) |
| `exodus-ad-01`         | Windows Server — **Active Directory / DNS** |

---

## Repository layout

```
ansible/
  ansible.cfg                     # config (inventory, roles path, vault password file)
  requirements.yml                # collection dependencies (community.general, pinned)
  inventories/production/
    hosts.yml                     # fleet inventory (functional + tier groups)
    group_vars/                   # group variables (incl. encrypted vault)
    host_vars/                    # per-host expected-services (health verification)
    archive/                      # prior inventory versions (retained, not deleted)
  roles/
    vector/                       # centralised logging agent (config-as-code, Vault)
    patching/                     # tiered security patching + reboot detection + error handling
    dns_resolver/                 # DNS remediation (nmcli, for AD-joined nodes)
  playbooks/
    maintenance/                  # patching pipeline, DNS fix, Vector deployment
    operational/                  # fleet health assessment, idempotency demo
docs/
  change-management/              # ITIL 4-aligned change-classification policy
  change-records/                 # CR-011 (IAM), CR-012 (AD/RBAC/SSH), CR-013 (automation)
  runbooks/                       # operational runbooks (e.g. disk-fill incident)
.github/ISSUE_TEMPLATE/           # Standard / Enterprise change-request templates
```

---

## Featured: the `vector` role (config-as-code + Vault)

A representative example of the approach. The role:

1. Deploys the Vector package repository, then installs a **pinned** version
   (consistent across the fleet — no version drift).
2. Renders the Vector config from a **Jinja2 template**, injecting the OpenSearch
   password from **Ansible Vault** — the template in Git holds only a placeholder,
   never the secret.
3. Enables the service and restarts it only when the config changes (handler).

```bash
# Onboard one node to centralised logging
ansible-playbook -i inventories/production/hosts.yml \
  playbooks/maintenance/vector-fleet.yml --limit exodus-platform-01

# Roll out / enforce across the whole fleet (idempotent)
ansible-playbook -i inventories/production/hosts.yml \
  playbooks/maintenance/vector-fleet.yml
```

Every node ships its logs to OpenSearch, configured entirely as code, with the
credential kept encrypted in Vault.

---

## Tech stack

**OS:** RHEL 9.2 · Windows Server (AD) · **Automation:** Ansible (ansible-core 2.14)
· **Secrets:** Ansible Vault · **Identity:** Active Directory (Kerberos, SSSD, RBAC) ·
**Logging:** Vector → OpenSearch · **Metrics:** Prometheus + node_exporter → Grafana
· **Alerting:** Alertmanager · **Ingress:** NGINX · **VCS / Governance:** Git +
GitHub (Issues, tiered change templates)

---

## Roadmap — done, and where it's heading

**Done**
- Ansible foundation, inventory design, control-node setup
- Cross-platform identity: RHEL fleet joined to Windows AD (Kerberos, SSSD, RBAC)
- Security hardening: IAM least-privilege (CR-011), SSH hardening fleet-wide (CR-012)
- Roles: `dns_resolver`, `patching` (tiered pipeline), `vector` (fleet-wide logging)
- Config-as-code centralised logging across the whole fleet (Vault-injected secrets)
- Ansible Vault + vault-password-file; credential rotation
- Change governance (CR-011 → CR-013), ITIL 4 classification policy, first runbook

**Next**
- **Onboarding capstone** — compose roles into one-command server onboarding (`site.yml`)
- Close out Phase 6 (automation foundation)
- **Application + database** (containerised) and application-layer observability
- **Kubernetes** — orchestrate the containerised workloads
- **Terraform** — infrastructure provisioning as code
- **CI/CD** — automated linting, testing, and deployment (GitHub Actions)
- **Cloud migration** — re-platform onto AWS (managed services, IAM, Secrets Manager)

---

## About

EXODUS is a personal, self-directed project — built to learn by doing and to
demonstrate real infrastructure- and platform-engineering practice on running
systems. Built and maintained by **Abiola Osota**
([github.com/Abiolathedon](https://github.com/Abiolathedon)).
