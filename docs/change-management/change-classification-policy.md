# EXODUS Change Classification Policy

**Policy Version:** 1.0
**Owner:** Platform Engineer (L3)
**Parent Change Request:** CR-013 — Phase 6 Automation Governance Foundation
**Governing Ticket:** CR-013.001 (#18)
**Aligned Standard:** ITIL 4 Change Enablement

---

## Purpose

This policy defines how every EXODUS engineering change is classified into
one of three tiers — **Standard**, **Normal**, or **Emergency** — so that the
amount of governance applied is matched to the risk of the change. This is the
ITIL 4 principle of *right-sized governance*: routine low-risk work stays fast,
significant work stays fully controlled, and urgent work has a safe expedited
path.

The tier is recorded in the **Change Tier** field of the Enterprise Engineering
Change Record (template v4.2+).

---

## The three change tiers

### 1. Standard Change

**Definition:** A pre-approved, low-risk, routine change that follows a known,
repeatable procedure. Because the procedure is understood and has been
pre-authorised, no separate per-instance approval is required.

**Characteristics:**
- Low risk, well-understood, reversible.
- Follows an established, documented method.
- Blast radius limited (single/few hosts, single service, or documentation).
- Belongs to the pre-approved standard-change catalogue (below).

**Record depth:** The Enterprise Engineering Change Record with only the
**mandatory core** fields completed. Elaborative fields (known-good comparison,
enterprise-standard explanations, interview learning, secondary impact
breakdowns) may be left blank.

**Approval:** Pre-approved by this policy. No separate approval needed.

**Examples in EXODUS:** DNS resolver correction, package install/removal via an
approved playbook, adding a monitoring target, routine documentation updates.

---

### 2. Normal Change

**Definition:** Any change that is not a pre-approved Standard change and is not
an Emergency. It requires assessment and approval before implementation.

**Characteristics:**
- Medium-to-high risk, or novel, or wide blast radius.
- No pre-approved procedure exists, or the change alters architecture/design.
- Needs full risk assessment, rollback design, and validation planning.

**Record depth:** The **full** Enterprise Engineering Change Record — core plus
the elaborative planning, decision, standards, and (where significant) interview
and impact-assessment fields.

**Approval:** Assessed and approved by the change decision owner before
implementation (in a real organisation, a change manager or Change Advisory
Board depending on risk).

**Examples in EXODUS:** Enterprise patch lifecycle rollout, onboarding
automation, application/database deployment, IaC provisioning, cloud migration,
changes to this change-management system itself.

---

### 3. Emergency Change

**Definition:** An urgent, high-impact change required to resolve a major
incident or close an active security threat, executed as fast as the risk
allows.

**Characteristics:**
- High urgency and high impact.
- Delay causes or prolongs harm (outage, data loss, active threat).
- Documentation is captured in real time as the work happens.

**Record depth:** The Enterprise Engineering Change Record, with the incident
context, actions, and evidence recorded live during and immediately after the
change. Planning fields are completed to the extent time allows, and completed
fully retrospectively.

**Approval:** Expedited authority (in a real organisation, an Emergency Change
Advisory Board or a single senior authority), with retrospective full review.

**Examples in EXODUS:** Emergency restoration of a failed critical service,
urgent remediation of an actively exploited vulnerability.

---

## How to classify a change (decision guide)

Ask, in order:

1. **Is it resolving a live major incident or active threat, right now, where
   waiting causes harm?** → **Emergency**.
2. **Is it in the pre-approved standard-change catalogue below (or equivalent:
   low-risk, routine, reversible, established procedure)?** → **Standard**.
3. **Otherwise** → **Normal** (assess and approve before implementing).

When in doubt between Standard and Normal, choose **Normal**. Over-classifying is
safe; under-classifying is not.

---

## Pre-approved Standard Change catalogue

The following change types are pre-approved as **Standard** when performed via
the established, governed method (GitOps flow, dry-run, verify):

| # | Standard Change | Method |
|---|-----------------|--------|
| S1 | DNS resolver correction (NetworkManager layer) | Ansible playbook/role |
| S2 | Package install or removal | Approved Ansible playbook/role |
| S3 | Add or update a monitoring target/exporter | Ansible + config template |
| S4 | Add or update a logging source | Ansible + config template |
| S5 | Routine documentation update | Direct commit |
| S6 | Non-behavioural config tidy-up (comments, formatting) | Reviewed commit |

Additions to this catalogue are themselves a **Normal** change to this policy.

---

## Relationship to the Enterprise Engineering Change Record

- Every change still uses the Enterprise Engineering Change Record template.
- The **Change Tier** field records the classification.
- Tier determines **how much** of the record is completed:
  - **Standard** → mandatory core only.
  - **Normal** → full record.
  - **Emergency** → live/retrospective full record.
- This gives one template, right-sized to the change — no separate lightweight
  form to maintain.

---

## Review

This policy is reviewed whenever the change-management system materially
changes. Amendments are made as a Normal change under governance.