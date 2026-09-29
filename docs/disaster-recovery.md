# Disaster Recovery

> Status: 📝 Planned (rehearsed in Phases 16, 19, and 21)

## What?

The procedure for rebuilding any node, or the whole environment, from a clean OS.

## Why?

The project's success criterion is reproducibility: after a total loss, the infrastructure returns using only Ansible, Git, Kubernetes, GitOps, and backups.

## Recovery Objectives (targets, to be measured)

| Metric | Target |
|--------|--------|
| RTO (time to restore) | To be measured during Phase 21 |
| RPO (acceptable data loss) | Equal to backup interval (daily baseline) |

## Prerequisites (must exist outside the failed node)

- Git repository (code and encrypted vault)
- Ansible Vault password (stored securely offline)
- Latest backups on an off-node location
- Backup encryption keys
- Provider console access and DNS control

## Scenario: VPS #2 Completely Destroyed

```text
1. Reinstall OS, then run bootstrap        (Ansible: bootstrap, users, security)
2. Re-join WireGuard                       (Ansible: wireguard)
3. Configure firewall and DNS              (Ansible: firewall, dns)
4. Install container runtime and K3s       (Ansible: docker, kubernetes)
5. Restore storage layer / NFS data        (Ansible: nfs, backup restore)
6. Restore PostgreSQL / MariaDB            (restore from latest verified backup)
7. Let GitOps reconcile applications       (gitops)
8. Redeploy monitoring, then verify alerts
9. Run end-to-end checks
```

## Scenario: VPS #1 (Edge) Destroyed

1. Rebuild OS and re-run bootstrap, security, and WireGuard.
2. Restore Nginx configuration and TLS via Ansible (certificates are re-issued by Let's Encrypt).
3. Reconnect peers and verify tunnels.

Note: public traffic is unavailable until VPS #1 returns, because it is a single edge node.

## Scenario: Laptop VM Lost

1. Recreate the VM, install Ansible, and clone the repository.
2. Restore the Vault password and SSH keys from secure storage.
3. Re-establish the WireGuard peer.

Production keeps running because the Laptop VM is not in the production traffic path.

## Verification After Recovery

```bash
ansible all -m ping
wg show
kubectl get nodes && kubectl get pods -A
# Application health endpoints
curl -fsS https://<domain>/health
# Data check: row counts / known records
# Monitoring: all Prometheus targets UP, test alert delivered
```

## Rehearsal Log

| Date | Scenario | Result | Time to recover | Issues found |
|------|----------|--------|-----------------|--------------|
| — | — | — | — | — |

## How to keep this accurate?

Update this document whenever a role, backup path, or dependency changes. The final rebuild in Phase 21 is the real test of this document.
