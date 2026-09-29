# Security

> Status: 📝 Planned (implemented mainly in Phase 6, applied across all phases)

## What?

Defense in depth: every layer restricts access independently.

| Layer | Control |
|-------|---------|
| Access | SSH key auth only, no root login, dedicated `devops` user with sudo |
| Network | Default-deny firewall, WireGuard private network, private DNS |
| Transport | TLS everywhere public (Let's Encrypt), automated renewal |
| Kubernetes | RBAC, NetworkPolicy (default deny in sensitive namespaces), resource limits |
| Containers | Non-root users, pinned image tags, minimal images, private registry |
| Secrets | Ansible Vault, Kubernetes Secrets, nothing plain in Git |
| Data | Encrypted backups, restricted database access |
| Detection | Log monitoring, alerting on auth failures and anomalies |

## Firewall Policy

```text
INPUT:   default DROP
FORWARD: controlled
OUTPUT:  controlled / allowed
```

Allowed public inbound: `22/tcp`, `80/tcp`, `443/tcp`, `51820/udp`. Any new port must be documented and justified before opening.

## SSH Hardening

```text
PasswordAuthentication no
PermitRootLogin no
```

**Safe procedure (avoids lockout):**
1. Create the `devops` user with an SSH key and sudo.
2. Open a **second** session and confirm login and sudo work.
3. Only then disable root and password authentication.
4. Keep the provider console available as a fallback.

## Secrets Management

Never committed: `.env`, passwords, private keys, API tokens, SSH keys, TLS private keys, database credentials, WireGuard private keys.

- Ansible: Ansible Vault (`ansible/vault/`)
- Kubernetes: Kubernetes Secrets (encrypted at rest or via a sealed-secrets approach, decided in Phase 9/13)
- Enforced by `.gitignore` and a pre-commit secret scan

## Kubernetes RBAC and NetworkPolicy

- Separate ServiceAccounts per workload, no use of `cluster-admin` for applications.
- Example allowed flow: `frontend → backend → PostgreSQL` and `backend → Redis`.
- Direct `frontend → PostgreSQL` is denied.

## Where?

- Ansible roles: `firewall`, `ssh`, `users`
- Kubernetes: `kubernetes/policies/`

## How to verify?

```bash
sshd -T | grep -E 'passwordauthentication|permitrootlogin'
sudo nft list ruleset            # or: sudo ufw status verbose
ss -tulpn                        # only expected listeners
nmap -Pn <public-ip>             # from outside: only allowed ports open
kubectl get networkpolicy -A
```

## How to recover?

- Locked out: provider web console, then revert the SSH or firewall change via Ansible.
- Suspected key leak: rotate keys, update Vault, re-run playbooks.
- Every firewall change is applied through Ansible with a tested rollback path.
