# Maktab Sharif DevOps — SRS Project

> An automation-first, secure, observable, recoverable, and reproducible infrastructure built on 2 VPS servers and 1 management VM.

![Status](https://img.shields.io/badge/status-in%20progress-yellow)
![IaC](https://img.shields.io/badge/IaC-Ansible-red)
![Kubernetes](https://img.shields.io/badge/Kubernetes-K3s-blue)
![Network](https://img.shields.io/badge/VPN-WireGuard-88171A)

## 🎯 Project Goal

Build a production-like infrastructure where **everything that can be expressed as code is managed as code**, not through manual commands:

```text
Manual (testing/debugging only) → Ansible / Kubernetes / GitOps → Final State
```

Success means: the service works **+** it is automated **+** secure **+** monitored **+** documented **+** recoverable **+** reproducible.

## 🏗️ Architecture

| Node | Specs | Role |
|------|-------|------|
| **VPS #1** | 6 GB RAM / 3 vCPU / 40 GB disk | Edge: firewall, WireGuard, Nginx reverse proxy, TLS, bastion |
| **VPS #2** | 6 GB RAM / 3 vCPU / 40 GB disk | Workload: K3s, PostgreSQL, Redis, registry, monitoring |
| **Laptop VM** | Ubuntu Server 24.04, 8 GB / 2 vCPU | Management: Ansible control node, Git/GitOps, no production traffic |

```text
                    INTERNET
                        |
         +--------------+--------------+
         |                             |
     VPS #1 (Edge)                VPS #2 (K3s)
     10.50.0.1  <--- WireGuard --->  10.50.0.2
         \
          \--------- 10.50.0.3 --------- Laptop VM (Ansible)
```

> ⚠️ K3s in this project is **non-HA** (a single control plane). IP addresses are examples and will be finalized after the infrastructure audit.

## 🧰 Tech Stack

- **Provisioning / Configuration:** Ansible (+ Ansible Vault)
- **Network & Security:** WireGuard, nftables/UFW (default deny), SSH hardening, private DNS (BIND9)
- **Orchestration:** K3s, Ingress, NetworkPolicy, RBAC, NFS storage
- **Data:** PostgreSQL (primary + backup), Redis, MariaDB (WordPress)
- **CI/CD & GitOps:** Gitea, Drone CI, private container registry, GitOps repository
- **Observability:** Prometheus, Grafana, Alertmanager, Loki/ELK (based on resource budget)
- **Edge:** Nginx, Let's Encrypt (auto-renewal), static caching
- **Backup:** Automated, encrypted, with retention policy and restore testing

## 📁 Repository Structure

```text
maktab-sharif-devops/
├── ansible/        # inventory, playbooks, roles, vault
├── kubernetes/     # namespaces, networking, storage, database, apps, monitoring, policies
├── gitops/         # deployment manifests
├── docs/           # architecture, network, security, kubernetes, backup, monitoring, DR
└── README.md
```

## 🗺️ Roadmap

| Phase | Title | Status |
|-------|-------|--------|
| 0 | Project Preparation | ⏳ |
| 1 | Infrastructure Audit | ⬜ |
| 2 | Clean State | ⬜ |
| 3 | Ansible Control Node | ⬜ |
| 4 | Base Server | ⬜ |
| 5 | Private Network (WireGuard) | ⬜ |
| 6 | Security | ⬜ |
| 7 | Private DNS | ⬜ |
| 8 | Container Runtime | ⬜ |
| 9 | Kubernetes (K3s) | ⬜ |
| 10 | Infrastructure Services | ⬜ |
| 11 | Applications | ⬜ |
| 12 | CI/CD | ⬜ |
| 13 | GitOps | ⬜ |
| 14 | Monitoring | ⬜ |
| 15 | Logging | ⬜ |
| 16 | Backup & Restore | ⬜ |
| 17 | Alerting | ⬜ |
| 18 | End-to-End Testing | ⬜ |
| 19 | Failure Testing | ⬜ |
| 20 | Documentation | ⬜ |
| 21 | Final Rebuild | ⬜ |

## 🔐 Security & Secrets

- No secrets (passwords, private keys, tokens, `.env` files) are ever committed to Git.
- Ansible secrets are encrypted with **Ansible Vault**.
- Public inbound ports are limited to: `22/tcp`, `80/tcp`, `443/tcp`, `51820/udp`.
- Administrative access goes through the WireGuard private IPs whenever possible.

## 🚀 Quick Start

```bash
git clone https://github.com/<username>/maktab-sharif-devops.git
cd maktab-sharif-devops/ansible
ansible all -i inventory/production/hosts.yml -m ping
```

## 🔁 Workflow per Phase

```text
Implement → Test → Verify → Document → Commit
```

Each phase gets its own commit following Conventional Commits, for example:
`feat(network): deploy wireguard`

## 📚 Documentation

| Document | Description |
|----------|-------------|
| [Architecture](docs/architecture.md) | Overall architecture |
| [Network](docs/network.md) | WireGuard, DNS, routing |
| [Security](docs/security.md) | Hardening, firewall, RBAC |
| [Kubernetes](docs/kubernetes.md) | K3s, Ingress, NetworkPolicy |
| [Backup](docs/backup.md) | Backup, retention, restore |
| [Monitoring](docs/monitoring.md) | Metrics, dashboards, alerts |
| [Disaster Recovery](docs/disaster-recovery.md) | Rebuilding from scratch |

## ✅ Success Criteria

The project is complete when the entire infrastructure can be rebuilt from a **clean OS** using only Ansible + Git + Kubernetes + GitOps, with no unnecessary manual intervention.

## 👤 Author

**<Your Name>** — Maktab Sharif DevOps Bootcamp
