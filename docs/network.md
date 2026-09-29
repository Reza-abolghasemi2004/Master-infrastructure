# Network

> Status: 📝 Planned (implemented in Phases 5 and 7)

## What?

A private WireGuard network connecting all three nodes, plus a private DNS zone for internal service names.

## Addressing (sample values, verified for conflicts in Phase 1)

| Node | WireGuard IP | Public IP |
|------|--------------|-----------|
| VPS #1 | 10.50.0.1 | yes |
| VPS #2 | 10.50.0.2 | yes |
| Laptop VM | 10.50.0.3 | none |

Network: `10.50.0.0/24`

## Topology (hub-and-spoke)

```text
Laptop VM (10.50.0.3) ---> VPS #1 (10.50.0.1, hub) <--- VPS #2 (10.50.0.2)
```

- VPS #1 is the hub because it is the only stable, reachable entry point besides VPS #2.
- The Laptop VM has no public IP, so it **initiates** the tunnel with `PersistentKeepalive = 25`.
- Traffic between the Laptop VM and VPS #2 is routed through VPS #1, which requires `net.ipv4.ip_forward=1` and matching firewall FORWARD rules (managed by Ansible).

## Why?

- Management traffic (SSH, Ansible, kubectl, metrics) stays off the public internet.
- Services such as PostgreSQL, Redis, registry, and monitoring are reachable only via private IPs.

## Public Ports (nothing else may be opened)

| Port | Purpose |
|------|---------|
| 22/tcp | SSH (restricted or closed after the WireGuard phase) |
| 80/tcp | HTTP (redirect and ACME) |
| 443/tcp | HTTPS |
| 51820/udp | WireGuard |

## Private DNS

- Server: BIND9 or a lightweight alternative (decided in Phase 7).
- Reachable only from the WireGuard network.
- Planned records:

| Name | Purpose |
|------|---------|
| git.internal.example | Gitea |
| registry.internal.example | Private registry |
| postgres.internal.example | PostgreSQL |
| redis.internal.example | Redis |
| k8s.internal.example | Kubernetes API |
| monitoring.internal.example | Grafana / Prometheus |

`internal.example` is a placeholder zone and will be replaced with the real internal domain.

## Where?

- Ansible role: `ansible/roles/wireguard/`, `ansible/roles/dns/`
- Playbooks: `ansible/playbooks/wireguard.yml`, `dns.yml`
- Private keys: Ansible Vault only, never in Git

## How to verify?

```bash
wg show                          # handshake recent, transfer counters increasing
ping 10.50.0.2                   # from Laptop VM
ip route                         # routes exist for 10.50.0.0/24
dig @10.50.0.1 git.internal.example   # DNS resolution (address depends on final DNS host)
```

## How to recover?

- Tunnel down: check `systemctl status wg-quick@wg0`, firewall rule for 51820/udp, and the keepalive on the Laptop VM.
- Lost keys: regenerate with the Ansible role and redistribute; keys are stored encrypted in Vault.
- Locked out of SSH: use the provider's web console (never close public SSH until private access is confirmed working).
