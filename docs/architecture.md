# Architecture

> Status: 📝 Planned (validated in Phase 0, finalized after Phase 1 audit)

## What?

A production-like infrastructure built on three nodes, managed entirely as code (Ansible, Kubernetes manifests, GitOps).

| Node | Specs | Role |
|------|-------|------|
| VPS #1 | 6 GB RAM / 3 vCPU / 40 GB disk, public IP | Edge: firewall, WireGuard hub, Nginx, TLS, bastion |
| VPS #2 | 6 GB RAM / 3 vCPU / 40 GB disk, public IP | Workload: K3s, PostgreSQL, Redis, registry, monitoring |
| Laptop VM | Ubuntu Server 24.04, 8 GB / 2 vCPU, no public IP | Management: Ansible control node, Git/GitOps |

```text
                        INTERNET
                            |
             +--------------+--------------+
             |                             |
        VPS #1 (Edge)                 VPS #2 (K3s)
        10.50.0.1   <-- WireGuard -->  10.50.0.2
             |
        10.50.0.3
        Laptop VM (Ansible / Git)
```

## Why?

| Decision | Reason |
|----------|--------|
| Ansible for all server config | Reproducibility; manual changes are debugging-only |
| WireGuard private network | Encrypted management plane; no admin services on public IPs |
| K3s instead of full Kubernetes | Low memory footprint (fits 6 GB nodes) |
| Laptop VM is management-only | Its uptime depends on a home laptop and internet, so it must not serve production traffic |
| Lightweight observability first | Prometheus + Grafana (+ Loki) fit the budget better than a default ELK |

## Design Constraints

- **Non-HA:** K3s runs with a single control plane. A two-node cluster is not highly available and is never described as such.
- **Disk:** 40 GB per VPS requires retention for logs, images, backups, and metrics from day one.
- **RAM:** 6 GB per VPS requires explicit resource requests and limits on every workload.

## Resource Budget (estimates, to be validated in Phase 1 and measured later)

| Component (VPS #2) | Target RAM |
|--------------------|-----------|
| OS + K3s control plane | ~1.2 GB |
| PostgreSQL | ~512 MB |
| MariaDB (WordPress) | ~256 MB |
| Redis | ~128 MB |
| Registry + Gitea | ~400 MB |
| Prometheus + Grafana + Alertmanager | ~1 GB |
| Logging (Loki or limited ELK) | 512 MB – 1.5 GB |
| Applications (backend, frontend, WordPress) | ~800 MB |

If the total exceeds the available budget, services are moved to VPS #1 or replaced with lighter alternatives. Logging is the first candidate for change.

## Where?

- Ansible code: `ansible/`
- Kubernetes manifests: `kubernetes/`
- GitOps manifests: `gitops/`
- Inventory and IP plan: `ansible/inventory/production/`

## How to verify?

```bash
ansible all -i inventory/production/hosts.yml -m ping   # from the Laptop VM
wg show                                                 # on each node
kubectl get nodes                                       # from a host with kubeconfig
```

## How to recover?

See [Disaster Recovery](disaster-recovery.md). The architecture is designed so each node can be rebuilt from a clean OS.
