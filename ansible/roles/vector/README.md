# Vector Role

Onboards a RHEL node to **EXODUS centralised logging**. The role installs
[Vector](https://vector.dev) (pinned version), deploys its configuration from a
template with the OpenSearch password injected from **Ansible Vault**, and
enables the service. Each node ships its `journald` logs to **OpenSearch** on
`exodus-logging-01`, tagged with the node's name.

This role brings a node from bare (no Vector) to fully shipping logs, and is
idempotent — re-running it enforces the desired state (version, config, service).

## Requirements

- RHEL 9 node reachable from the control node.
- Internet access to `yum.vector.dev` (the Vector package repo).
- An encrypted Ansible Vault providing `opensearch_password` (referenced via
  `group_vars/all/vars.yml` -> `group_vars/all/vault.yml`). Run playbooks with
  `--ask-vault-pass` (or a vault password file).
- OpenSearch reachable at the configured endpoint.

## Role Variables

Defined in `defaults/main.yml` (override as needed):

| Variable | Default | Purpose |
|---|---|---|
| `vector_version` | `"0.58.0"` | Pinned Vector version (keeps the fleet consistent). |
| `vector_opensearch_endpoint` | `"https://192.168.0.156:9200"` | OpenSearch endpoint logs are shipped to. |
| `vector_index` | `"exodus-logs-vector"` | OpenSearch index logs are written to. |
| `vector_opensearch_user` | `"admin"` | OpenSearch user Vector authenticates as. |

The OpenSearch **password** is NOT a role variable — it is injected from the
Vault as `opensearch_password` (never stored in the repo).

Per-node values used by the template:
- `inventory_hostname` -> tags each log with the source node (`exodus_node`).

## Dependencies

None.

## Example Playbook

Apply the role via `playbooks/maintenance/vector-fleet.yml`:

```yaml
- name: Deploy Vector for centralised logging
  hosts: linux
  become: true
  roles:
    - vector
```

Run it (staged rollout with `--limit`):

```bash
# One node first
ansible-playbook -i inventories/production/hosts.yml \
  playbooks/maintenance/vector-fleet.yml \
  --limit exodus-platform-01 --ask-vault-pass

# Whole fleet
ansible-playbook -i inventories/production/hosts.yml \
  playbooks/maintenance/vector-fleet.yml --ask-vault-pass
```

## License

MIT

## Author Information

Abiola Osota — EXODUS home lab (github.com/Abiolathedon).