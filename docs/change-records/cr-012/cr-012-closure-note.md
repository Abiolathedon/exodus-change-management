# CR-012 — Fleet-Wide Linux Identity Rollout
# Enterprise Operational Remediation, Deterministic Parity Engineering, and Fleet Runtime Standardization

## Executive Summary

CR-012 evolved significantly beyond a standard Linux Active Directory rollout task and became a major enterprise operational remediation and governance-evolution phase inside the EXODUS v6 Enterprise Engineering Simulation.

The original objective of CR-012 was to safely expand previously validated Linux Active Directory integration and RBAC controls from a single validated node into the wider EXODUS Linux infrastructure fleet.

Although realm joins, Kerberos integration, SSSD installation, Active Directory group mapping, and identity resolution were already functioning correctly, interactive SSH authentication using centralized Active Directory identities consistently failed across most fleet nodes.

Through deterministic runtime parity engineering methodology, the root causes were eventually identified as historical SSH hardening drift inherited from earlier local-only authorization policies that directly conflicted with the newer centralized Active Directory architecture introduced during Phase 5 IAM expansion.

CR-012 ultimately resulted in successful fleet-wide restoration of centralized Active Directory SSH authentication while preserving hardened enterprise SSH runtime posture and introducing permanent new governance methodology based on authoritative runtime parity engineering.

---

# Infrastructure Scope

## Linux Infrastructure Fleet Included

- exodus-services-01
- exodus-observability-01
- exodus-grafana-01
- exodus-ingress-01
- exodus-logging-01

---

# Original Rollout Objective

The original objective of CR-012 was to expand previously validated Linux Active Directory integration from the pilot validation stage into the wider Linux fleet.

Target operational outcomes included:

- centralized Active Directory authentication
- centralized RBAC authorization
- AllowGroups-based SSH governance
- SSSD operationalization
- Kerberos-backed authentication
- governed SSH hardening preservation
- deterministic fleet standardization

---

# Initial Operational Symptoms

## Interactive Active Directory SSH Failures

The following fleet nodes consistently failed interactive Active Directory SSH authentication:

- exodus-observability-01
- exodus-grafana-01
- exodus-ingress-01
- exodus-logging-01

However, multiple lower-level identity systems already appeared healthy.

The following validations successfully passed during early troubleshooting:

- realm join validation
- Kerberos membership validation
- SSSD installation validation
- DNS validation
- Active Directory group resolution
- NSS identity resolution
- AllowGroups RBAC configuration
- PAM integration presence

This created an initially confusing operational state where identity resolution succeeded but SSH login authorization still failed.

---

# Authoritative Reference Node Methodology

## Operational Source Of Truth

During CR-012, the following node became the authoritative operational reference baseline:

- exodus-services-01

This node had already successfully validated:

- interactive Active Directory SSH login
- Kerberos-backed authentication
- SSSD functionality
- NSS resolution
- PAM behavior
- AllowGroups RBAC enforcement
- post-validation SSH re-hardening

Instead of continuing reactive troubleshooting methodology, the project transitioned into deterministic parity engineering methodology using exodus-services-01 as the runtime source of truth.

All future remediation decisions during CR-012 were validated directly against the runtime behavior of this node before execution.

This became one of the most important governance evolutions introduced during the project.

---

# Deterministic Runtime Parity Engineering

CR-012 fundamentally changed operational troubleshooting methodology inside EXODUS.

The project transitioned away from reactive trial-and-error troubleshooting and into deterministic runtime parity engineering methodology.

This methodology introduced:

- authoritative runtime comparison
- one-layer-at-a-time troubleshooting
- staged parity validation
- rollback-first remediation
- syntax-validation-before-reload governance
- runtime-state verification before assumptions
- deterministic remediation sequencing

This prevented unnecessary rebuilding of already healthy infrastructure layers.

---

# Runtime SSH Parity Discovery

## Historical SSH Hardening Drift

Deterministic runtime comparison between exodus-services-01 and failing nodes identified the following critical authorization drift:

```text
AllowUsers exodus-admin
```

located inside:

```text
/etc/ssh/sshd_config.d/60-allowusers.conf
```

Affected nodes included:

- exodus-observability-01
- exodus-grafana-01
- exodus-ingress-01
- exodus-logging-01

The authoritative working node did not contain this runtime authorization restriction.

---

# Historical Drift Analysis

The AllowUsers configuration originated from historical Phase 1 SSH hardening policies that were designed around local Linux-only administrative access before centralized Active Directory integration existed inside EXODUS.

At the time of the original Phase 1 hardening work, restricting SSH access to the local exodus-admin account was considered correct security posture.

However, during later Phase 5 IAM evolution, the infrastructure transitioned into centralized Active Directory authorization architecture using:

- SSSD
- Kerberos
- AllowGroups
- centralized RBAC
- Active Directory security groups

The historical local-only SSH authorization model directly conflicted with the newer centralized authorization architecture.

This became the primary root cause of the fleet-wide Active Directory SSH authentication failures.

---

# SSH Runtime Evidence

Runtime SSH validation confirmed affected nodes contained:

```text
allowusers exodus-admin
```

while the authoritative reference node did not.

SSH authentication logs later confirmed the exact runtime failure condition:

```text
not allowed because not listed in AllowUsers
```

This was one of the most important discoveries during CR-012 because it conclusively proved the issue was NOT caused by:

- Kerberos failure
- DNS failure
- realm join failure
- SSSD installation failure
- PAM installation failure
- identity resolution failure

The failure existed specifically inside the SSH authorization layer.

---

# Incorrect Early Assumptions Prevented

The deterministic parity methodology prevented several dangerous incorrect remediation paths.

Without runtime comparison against the authoritative working node, troubleshooting could have incorrectly drifted into:

- unnecessary realm re-joins
- Kerberos rebuilds
- SSSD reinstalls
- PAM reconstruction
- DNS redesign
- Active Directory rollback operations

Because parity analysis proved those layers already matched the working node, the project avoided destructive unnecessary remediation.

This represented a major operational maturity evolution inside EXODUS.

---

# One-Layer-At-A-Time Troubleshooting

CR-012 introduced strict staged troubleshooting methodology.

Validation and remediation were deliberately separated into the following controlled operational stages:

1. SSH runtime parity validation
2. AllowUsers runtime drift identification
3. runtime configuration source tracing
4. rollback snapshot creation
5. rollback-safe configuration backup
6. AllowUsers runtime drift removal
7. SSH syntax validation
8. SSH runtime reload validation
9. runtime parity revalidation
10. SSSD parity validation
11. SSSD runtime drift remediation
12. SSSD restart validation
13. identity resolution validation
14. localhost Active Directory login validation
15. login-format validation
16. SSH re-hardening
17. final runtime parity validation

This structured methodology dramatically reduced operational ambiguity and eliminated blind troubleshooting behavior.

---

# Fleet-Wide Drift Confirmation

After successful remediation of exodus-observability-01, deterministic runtime comparison revealed the SAME exact drift pattern existed identically across:

- exodus-grafana-01
- exodus-ingress-01
- exodus-logging-01

This transformed the issue from isolated server drift into confirmed fleet-wide historical configuration drift.

The observability remediation sequence then became the validated remediation model for the remaining fleet rollout.

---

# SSSD Runtime Drift Discovery

Additional parity analysis identified a second deterministic runtime drift.

The following SSSD configuration existed on the authoritative working node but was missing on affected nodes:

```ini
ad_gpo_access_control = permissive
```

inside:

```text
/etc/sssd/sssd.conf
```

Affected nodes:

- exodus-observability-01
- exodus-grafana-01
- exodus-ingress-01
- exodus-logging-01

This runtime parity mismatch was safely remediated after rollback-safe backups were created.

---

# Incorrect Authentication Test Discovery

During localhost SSH validation, early failed login attempts initially appeared to indicate broader authentication failure.

Deterministic journal analysis later revealed the actual issue was incorrect login formatting.

Incorrect login test:

```text
exodus-user
```

Correct realm-qualified login format:

```text
exodus-user@exodus.local
```

This discovery prevented unnecessary:

- Kerberos rebuild operations
- SSSD reinstall attempts
- realm leave/rejoin operations
- PAM reconstruction attempts

and further reinforced the value of deterministic parity engineering methodology.

---

# Runtime Validation Methodology

CR-012 heavily emphasized runtime-state validation instead of configuration-file assumptions.

Validation included:

- sshd runtime state validation
- SSSD runtime validation
- identity resolution validation
- journal log analysis
- SSH runtime comparison
- localhost authentication testing
- interactive shell validation
- final runtime parity confirmation

This established a much more mature operational engineering model inside EXODUS.

---

# Snapshot And Rollback Governance

Rollback-first governance remained mandatory throughout CR-012 execution.

VMware snapshots were created before remediation for:

- exodus-services-01
- exodus-observability-01
- exodus-grafana-01
- exodus-ingress-01
- exodus-logging-01

Additional rollback-safe configuration backups were also created before all:

- SSH modifications
- SSSD modifications
- runtime remediation operations

This preserved deterministic rollback capability throughout the remediation process.

---

# Interactive Active Directory Login Validation

Interactive Active Directory SSH authentication was successfully validated across the fleet using the correct realm-qualified login format:

```text
exodus-user@exodus.local
```

Successful validation occurred on:

- exodus-observability-01
- exodus-grafana-01
- exodus-ingress-01
- exodus-logging-01

This confirmed successful centralized Active Directory authorization functionality fleet-wide.

---

# SSH Re-Hardening Phase

During validation stages, temporary runtime validation mode used:

```text
PasswordAuthentication yes
```

After successful login validation completed, enterprise SSH hardening was restored fleet-wide using:

```text
PasswordAuthentication no
```

This ensured the infrastructure returned to hardened runtime posture after deterministic validation completed.

---

# Final Hardened Runtime Parity

Final runtime parity validation confirmed all fleet nodes matched the authoritative hardened SSH baseline.

Final validated runtime state:

```text
usepam yes
passwordauthentication no
allowgroups sg-linux-access@exodus.local
allowgroups wheel
```

Legacy runtime authorization drift:

```text
AllowUsers exodus-admin
```

was fully removed from runtime authorization control.

---

# Governance Evolution

CR-012 permanently changed future EXODUS operational governance.

The following governance principles are now permanently enforced:

- authoritative reference-node methodology
- deterministic parity engineering
- runtime behavior comparison before remediation
- no blind fleet rollout
- rollback-first methodology
- syntax-validation-before-reload governance
- validation-before-hardening
- one-layer-at-a-time troubleshooting
- runtime-state verification before assumptions
- parity-first operational engineering

---

# Phase 6 Future Automation Impact

CR-012 established permanent future governance requirements for Phase 6 automation and future Ansible onboarding workflows.

Future server onboarding must now include deterministic parity validation against authoritative operational reference nodes before production rollout.

Future onboarding validation must include parity checks for:

- SSH runtime state
- SSSD configuration
- PAM behavior
- Kerberos behavior
- realm configuration
- ingress integration
- observability integration
- centralized logging integration
- RBAC validation
- hardened runtime configuration

Blind automation rollout methodology is now permanently prohibited inside EXODUS governance.

---

# Final Operational Outcome

CR-012 successfully completed:

- fleet-wide Linux Active Directory remediation
- runtime SSH authorization standardization
- deterministic fleet parity restoration
- centralized RBAC operationalization
- enterprise SSH hardening restoration
- fleet-wide runtime parity standardization
- deterministic operational governance evolution

The EXODUS Linux infrastructure fleet now operates using centralized Active Directory authorization with deterministic runtime parity engineering methodology and restored enterprise hardened SSH posture.