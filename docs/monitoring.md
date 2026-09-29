# Monitoring

> Status: 📝 Planned (implemented in Phases 14, 15, and 17)

## What?

Metrics, dashboards, and alerts covering hosts, Kubernetes, data services, Nginx, domains, and certificates.

## Stack (planned)

| Function | Tool |
|----------|------|
| Metrics | Prometheus |
| Dashboards | Grafana |
| Alerts | Alertmanager (Email / Slack / RocketChat) |
| Host metrics | node_exporter |
| Kubernetes metrics | kube-state-metrics, cAdvisor |
| PostgreSQL / Redis | postgres_exporter, redis_exporter |
| Nginx | nginx exporter plus access-log parsing |
| Domains and TLS | blackbox_exporter (HTTP, DNS, certificate expiry) |
| Logs | Loki, or ELK if the resource budget allows |

## Why?

You cannot operate or defend what you cannot see. The stack is chosen to fit 6 GB nodes.

## Coverage

| Area | Signals |
|------|---------|
| VM | CPU, RAM, disk, load, network, disk IO |
| Domain | HTTP, HTTPS, DNS, certificate expiry, response time |
| PostgreSQL | Connections, status, replication (if any), disk |
| Redis | Availability, memory, connections |
| Kubernetes | Nodes, pods, deployments, CPU, memory, restarts, PVC |
| Nginx | Requests, errors, traffic, response time, upstream time, cache status |

## Nginx: Latest 50 Requests

Access logs use a custom format including URI, method, status, request time, upstream time, cache status, and client. Logs are shipped to the logging backend and a dashboard panel or saved query shows the latest 50 requests.

## Alerts

| Alert | Condition (example) |
|-------|---------------------|
| CPU High | > 85% for 10 min |
| RAM High | > 90% for 10 min |
| Disk Almost Full | > 85% used |
| Node Down | target unreachable for 2 min |
| Pod CrashLoop | restart count rising |
| Certificate Expiry | < 14 days |
| PostgreSQL Down | exporter reports down |
| Redis Down | exporter reports down |
| Backup Failed | last success older than 26 hours |
| Nginx Error Rate High | 5xx ratio above threshold |
| Domain Down | probe failing for 2 min |

Thresholds are starting points and are tuned after observing normal behavior.

## Retention

- Prometheus: limited by time and size (for example 7 to 15 days, capped in GB).
- Logs: about 7 days, adjusted to real disk usage.
- Journald and Docker logs: size-limited with rotation.

## Where?

- Ansible role: `ansible/roles/monitoring/`
- Kubernetes: `kubernetes/monitoring/`
- Access: private network only (`monitoring.internal.example`)

## How to verify?

```bash
kubectl get pods -n monitoring
curl -s http://<prometheus>:9090/-/healthy
# Prometheus UI: Status → Targets (all UP)
# Trigger a test alert and confirm it arrives in the notification channel
```

## How to recover?

- Dashboards and alert rules are stored as code, so redeploying restores them.
- Metric history is not critical and may be lost. Configuration must never be.
