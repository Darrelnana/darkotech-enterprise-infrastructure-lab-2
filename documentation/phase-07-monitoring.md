# Phase 7 — Monitoring: Prometheus, Grafana, Alerting

**Status:** 🔧 In Progress
**Date started:** March 2026

---

## Objective

Deploy centralized monitoring across all lab VMs using Prometheus for metrics collection and Grafana for visualization. Configure alerting thresholds and centralized syslog so all system events flow to one place.

---

## Architecture

```
All VMs (node_exporter)
        │
        │ metrics (port 9100)
        ▼
Ubuntu-Srv (Prometheus)
        │
        │ data source
        ▼
Ubuntu-Srv (Grafana :3000)
        │
        │ reverse proxy
        ▼
NGINX (port 80)
```

---

## Components

| Component | Host | Port | Purpose |
|---|---|---|---|
| Prometheus | Ubuntu-Srv | 9090 | Metrics scraping and storage |
| Grafana | Ubuntu-Srv | 3000 | Dashboard and visualization |
| Node Exporter | All VMs | 9100 | System metrics per host |
| NGINX | Ubuntu-Srv | 80 | Reverse proxy for Grafana |

---

## Prometheus Configuration

Prometheus scrapes metrics from node_exporter on each VM every 15 seconds.

Targets configured in `/etc/prometheus/prometheus.yml`:

```yaml
scrape_configs:
  - job_name: 'ubuntu-srv'
    static_configs:
      - targets: ['10.10.20.20:9100']

  - job_name: 'dc01'
    static_configs:
      - targets: ['10.10.20.10:9100']

  - job_name: 'dc02'
    static_configs:
      - targets: ['10.10.20.11:9100']
```

---

## Grafana Dashboards

### Node Exporter Full Dashboard
- CPU usage per VM
- Memory usage per VM
- Disk I/O
- Network traffic in/out
- System load average
- Uptime

### Alert Rules
| Alert | Threshold | Severity |
|---|---|---|
| High CPU | >85% for 5 minutes | Warning |
| Low disk space | <10% free | Critical |
| High memory | >90% for 5 minutes | Warning |
| Host down | No metrics for 1 minute | Critical |

---

## Monitoring Topics Covered

### A+ Topics
- System performance monitoring
- Baseline establishment
- Threshold alerting
- Log analysis and review

### CCNA Topics
- SNMP concepts (Prometheus is a modern alternative)
- Syslog protocol and centralization
- Network performance metrics
- Bandwidth monitoring

### Security+ Topics
- Continuous monitoring
- Anomaly detection via alerting
- Log aggregation for security analysis
- Incident detection through metric spikes

---

## Interview Talking Points

**"How do you monitor your lab environment?"**
> I deployed Prometheus on Ubuntu Server to scrape metrics from node_exporter running on every VM. Grafana pulls from Prometheus and displays CPU, memory, disk, and network dashboards per host. I set up alert rules so I get notified when CPU stays above 85% for 5 minutes or disk space drops below 10%. I can show you the dashboards live.

**"What would you look for in Grafana if a server was running slow?"**
> First I'd check CPU usage — is it pegged at 100%? Then memory — is the system swapping? Then disk I/O — high wait times suggest a storage bottleneck. Network graphs would show if there's unexpected traffic. The combination of all four metrics usually points to the root cause within a couple of minutes.
