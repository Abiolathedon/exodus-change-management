# INCIDENT RECORD / RUNBOOK — services-01 Disk Fill (rsyslog->OpenSearch cascade)

**Type:** Emergency Change (acted first, documented after — disk at 95% climbing)
**Parent Change Request:** CR-013 — Phase 6 Automation Governance Foundation
**Date:** 2026-09-16 / 2026-09-17
**Node affected:** exodus-services-01 (192.168.0.152)
**Severity:** High (root disk 95%, climbing toward 100% = service-breaking)
**Status:** RESOLVED
**Engineer:** Abiola Osota — Platform Engineer (L3)

---

## 1. Summary

The root filesystem on exodus-services-01 filled to 95% (from a healthy level),
risking a full-disk outage. Root cause: a broken rsyslog->OpenSearch output
(`searchType="_doc"`) that modern OpenSearch rejects, writing an 18 GB error
file. The full disk had also cascaded to crash the intended log shipper
(Vector). Remediated by disabling the broken rsyslog path, reclaiming the disk,
and restoring Vector. Logging consolidated onto Vector (the intended shipper).

---

## 2. Detection

Spotted via a Grafana dashboard gauge: "Root FS" showing ~92% (red) on
services-01 during routine observability review. (Note: there was NO disk-space
alert configured — detection was by chance. See follow-ups.)

---

## 3. Diagnosis (read-only investigation)

Confirm the disk usage:
```
ansible exodus-services-01 -i inventories/production/hosts.yml \
  -m command -a "df -h /" --become
# -> /dev/mapper/rhel-root 35G 32G 2.8G 93% /  (later 95%)
```

Find the biggest top-level directories:
```
ansible exodus-services-01 -i inventories/production/hosts.yml \
  -m shell -a "du -h --max-depth=1 / 2>/dev/null | sort -rh | head -12" --become
# -> /var = 22G  (biggest)
```

Drill into /var, then /var/log:
```
ansible exodus-services-01 -i inventories/production/hosts.yml \
  -m shell -a "du -h --max-depth=1 /var 2>/dev/null | sort -rh | head -12" --become
# -> /var/log = 21G

ansible exodus-services-01 -i inventories/production/hosts.yml \
  -m shell -a "ls -lhS /var/log/ | head -15" --become
# -> rsyslog-opensearch-error.json = 18G (actively growing)
#    lastlog = 11G (SPARSE file - not real disk use)
```

Read the error to find WHY shipping fails:
```
ansible exodus-services-01 -i inventories/production/hosts.yml \
  -m shell -a "tail -c 2000 /var/log/rsyslog-opensearch-error.json" --become
# -> OpenSearch reply: 400 "Action/metadata line [1] contains an unknown
#    parameter [_type]"  <-- ROOT CAUSE
```

Locate + read the rsyslog OpenSearch config:
```
ansible exodus-services-01 -i inventories/production/hosts.yml \
  -m shell -a "grep -rl 'opensearch\|_type\|exodus-logs' /etc/rsyslog.conf /etc/rsyslog.d/ 2>/dev/null" --become
# -> /etc/rsyslog.d/10-opensearch.conf

ansible exodus-services-01 -i inventories/production/hosts.yml \
  -m command -a "cat /etc/rsyslog.d/10-opensearch.conf" --become
# -> action(type="omelasticsearch" ... searchType="_doc" ...)  <-- the _type source
#    (also revealed a PLAINTEXT password in the config - security issue)
```

Check the intended shipper (Vector) - found FAILED:
```
ansible exodus-services-01 -i inventories/production/hosts.yml \
  -m shell -a "systemctl status vector --no-pager -l | head -30" --become
# -> failed, ExecStartPre vector validate exited status=78/CONFIG,
#    restart-looped 12x, down ~24h (crashed when the disk filled)
```

---

## 4. Root Cause

- Modern OpenSearch removed the `_type` parameter. The rsyslog omelasticsearch
  output in /etc/rsyslog.d/10-opensearch.conf still sent `searchType="_doc"`,
  so EVERY log was rejected (HTTP 400) and logged to an error file -> 18 GB.
- The full disk then cascaded: it crashed Vector (the intended, chosen shipper),
  so BOTH shippers were down and the disk kept filling.
- Architecture note: the archive shows rsyslog->OpenSearch was already found
  unreliable and Vector chosen instead. The rsyslog path was a broken,
  redundant leftover.

---

## 5. Remediation (fix, in order — stop source, reclaim, restore)

Stop the source: disable the broken rsyslog OpenSearch output (rename, keep for reference):
```
ansible exodus-services-01 -i inventories/production/hosts.yml \
  -m command -a "mv /etc/rsyslog.d/10-opensearch.conf /etc/rsyslog.d/10-opensearch.conf.disabled" --become
```

Restart rsyslog so it drops the disabled output:
```
ansible exodus-services-01 -i inventories/production/hosts.yml \
  -m systemd -a "name=rsyslog state=restarted" --become
```

Reclaim disk: truncate the 18 GB error file (truncate, not delete - frees space in place):
```
ansible exodus-services-01 -i inventories/production/hosts.yml \
  -m shell -a "truncate -s 0 /var/log/rsyslog-opensearch-error.json" --become
```

Verify disk recovered:
```
ansible exodus-services-01 -i inventories/production/hosts.yml \
  -m command -a "df -h /" --become
# -> 35G 13G 22G 37% /   (95% -> 37%, ~20 GB reclaimed)
```

Restore the intended shipper (Vector) - config now validates once disk freed:
```
ansible exodus-services-01 -i inventories/production/hosts.yml \
  -m command -a "vector validate" --become
# -> Validated OK (Health check "exodus_opensearch" passed; 2 TLS warnings)

ansible exodus-services-01 -i inventories/production/hosts.yml \
  -m systemd -a "name=vector state=started" --become

ansible exodus-services-01 -i inventories/production/hosts.yml \
  -m command -a "systemctl is-active vector" --become
# -> active (stable, did not re-crash)
```

---

## 6. Validation (proof logging restored)

```
# (credential-exposing check - do NOT repeat; use Vault next time)
curl -s -k -u admin:<REDACTED> https://192.168.0.156:9200/_cat/indices/exodus-logs?v
# -> yellow open exodus-logs ... docs.count 78 ... 33.4kb
#    Logs ARE arriving in OpenSearch via Vector. (yellow = normal single-node.)
```

---

## 7. Outcome / Architecture Improvement

- Disk: 95% -> 37%, stable, will not refill (source stopped).
- Logging consolidated on VECTOR (single intended shipper); the broken,
  redundant rsyslog->OpenSearch path retired (left disabled).
- rsyslog still runs for LOCAL system logging (its normal job).

---

## 8. Follow-ups (added to improvement register)

- SECURITY (HIGH): rotate the OpenSearch admin password (exposed in plaintext
  config, terminal history, and chat); move to Ansible Vault; enable Vector
  verify_certificate / verify_hostname (currently disabled).
- Add DISK-SPACE alert (>85%) - this incident was found by chance, not alerted.
- Add RETENTION policy (logs/metrics) - no data-lifecycle discipline caused the fill.
- Config-as-code: manage rsyslog/Vector config via Ansible (was hand-placed).
- Delete the broken .disabled rsyslog config once satisfied (or Ansible-manage its absence).
- NOTE: a pre-incident VMware snapshot of services-01 is retained deliberately
  for a future disk-full / storage troubleshooting LEARNING exercise.

---

## 9. Lessons (interview-ready)

- Host disk fill can CASCADE (a full disk crashed the log shipper) - fix the
  source before reclaiming space, or it refills.
- "Service up" != "service healthy" != "service functional" - verified at every
  level (service active, config validates, health check, actual doc count).
- Deprecated API fields (`_type`) after a backend version change silently break
  pipelines - version-drift between shipper and backend.
- Separate OS/local logging (rsyslog) from log-aggregation shipping (Vector);
  don't run two shippers to the same destination.
- Monitoring must be PROACTIVE (disk alert) not reactive (found by chance).
