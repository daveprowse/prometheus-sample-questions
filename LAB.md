# Prometheus (PCA) Sample Questions Lab

A hands-on lab covering all 5 question topics.

See the corresponding YouTube video with 5 sample questions here: https://youtu.be/XSvS2zQdH7g

©️ Dave Prowse

---

## Requirements

- Two Debian-based Linux VMs (VirtualBox or similar)
  - **VM1:** Prometheus server + Alertmanager
  - **VM2:** Node Exporter (monitored target)
- Internet access to download binaries

> **Note:** Node Exporter can also run on VM1 for a simpler single-VM setup.
> The two-VM setup is recommended for a realistic lab experience.

---

## Lab Structure

```
prometheus-sample-questions/
├── LAB.md               # This file
├── prometheus.yml       # Prometheus configuration
└── alert_rules.yml      # Alerting rules
```

---

## How to Run

Follow each section in order. Each section corresponds to one sample question.

---

## Q1 — Binary Installation (Prometheus)

**On VM1:**

```bash
# Get the latest download URL from the official downloads page:
# https://prometheus.io/download/
# Copy the Linux amd64 link and paste below

wget <paste-prometheus-download-URL-here>
tar xvf prometheus-*.tar.gz
cd prometheus-*/

# Start Prometheus with default config
./prometheus --config.file=prometheus.yml
```

Verify Prometheus is running — open a browser on VM1:

```
http://localhost:9090
```

---

## Q2 — Node Exporter Installation and Scraping

**On VM2 (or VM1 for single-VM setup):**

```bash
# Get the latest download URL from the official downloads page:
# https://prometheus.io/download/#node_exporter
# Copy the Linux amd64 link and paste below

wget <paste-node_exporter-download-URL-here>
tar xvf node_exporter-*.tar.gz
cd node_exporter-*/

# Start Node Exporter
./node_exporter
```

Verify Node Exporter is exposing metrics on port 9100:

```bash
curl http://localhost:9100/metrics
```

**Configure Prometheus to scrape Node Exporter.**
Edit `prometheus.yml` on VM1 and add the following under `scrape_configs`:

```yaml
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node'
    static_configs:
      - targets: ['<VM2-IP>:9100']
```

> Replace `<VM2-IP>` with the IP address of VM2.
> For single-VM setup use `localhost:9100`.

Restart Prometheus and verify the node target appears as UP at:

```
http://localhost:9090/targets
```

---

## Q3 — Basic PromQL in the Web UI and with curl

**In the Prometheus Web UI (http://localhost:9090):**

Run the following queries in the expression browser:

```promql
# Q3: Check scrape target status — 1 = UP, 0 = DOWN
up

# See all node metrics
node_cpu_seconds_total
```

**With curl in the terminal on VM1:**

```bash
# Query the up metric via Prometheus HTTP API
curl 'http://localhost:9090/api/v1/query?query=up'

# Pretty print with jq if available
curl -s 'http://localhost:9090/api/v1/query?query=up' | jq .
```

---

## Q4 — Alertmanager Installation and Configuration

**On VM1:**

```bash
# Get the latest download URL from the official downloads page:
# https://prometheus.io/download/#alertmanager
# Copy the Linux amd64 link and paste below

wget <paste-alertmanager-download-URL-here>
tar xvf alertmanager-*.tar.gz
cd alertmanager-*/

# Start Alertmanager
./alertmanager --config.file=alertmanager.yml
```

Verify Alertmanager is running at:

```
http://localhost:9093
```

**Configure Prometheus to send alerts to Alertmanager.**
Edit `prometheus.yml` on VM1:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['localhost:9093']

rule_files:
  - "alert_rules.yml"

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'node'
    static_configs:
      - targets: ['<VM2-IP>:9100']
```

**Create `alert_rules.yml`:**

```yaml
groups:
  - name: basic_alerts
    rules:
      - alert: TargetDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Target {{ $labels.instance }} is down"
```

Restart Prometheus. Check alert rules are loaded at:

```
http://localhost:9090/rules
```

Check alert status at:

```
http://localhost:9090/alerts
```

> To test the alert: stop Node Exporter on VM2 and wait for the `for` duration to elapse.
> Watch the alert transition from `inactive` → `pending` → `firing`.

---

## Q5 — Intermediate PromQL: CPU Usage with Labels

**In the Prometheus Web UI or with curl:**

```promql
# Total CPU rate for all modes on a specific instance
rate(node_cpu_seconds_total{instance="<VM2-IP>:9100"}[5m])

# CPU usage excluding idle — the correct pattern
sum(rate(node_cpu_seconds_total{instance="<VM2-IP>:9100", mode!="idle"}[5m]))

# Break it down by CPU mode
sum by (mode) (rate(node_cpu_seconds_total{instance="<VM2-IP>:9100"}[5m]))
```

> Replace `<VM2-IP>` with your actual VM2 IP address.
> For single-VM setup use `localhost:9100`.

Observe how filtering by `mode!="idle"` changes the result compared to including all modes.

---

## What Each Step Demonstrates

| Step | Question Topic | What to Observe |
|------|---------------|-----------------|
| Q1 | Binary Installation | Prometheus starts and Web UI available at port 9090 |
| Q2 | Node Exporter | Target appears as UP in Prometheus at port 9100 |
| Q3 | Basic PromQL | `up` metric returns 1 for healthy targets |
| Q4 | Alerting | Alert transitions from pending → firing when target goes down |
| Q5 | PromQL with Labels | `mode!="idle"` correctly isolates CPU usage |

---

## Deliberate Break Exercise

After completing the lab, try these to reinforce exam concepts:

1. **Stop Node Exporter on VM2** — watch the `up` metric drop to `0` in the Web UI
2. **Wait for the `for` duration** — observe the alert move from `pending` to `firing`
3. **Restart Node Exporter** — watch the alert resolve automatically
4. **Run a bad PromQL query** — try `rate(up[5m])` and observe why it returns unexpected results (`up` is a Gauge, not a Counter)
5. **Check promtool:** Run `./promtool check config prometheus.yml` to validate your config

---

## Notes

- Prometheus default port: **9090**
- Node Exporter default port: **9100**
- Alertmanager default port: **9093**
- All binaries are single static executables — no package manager required
- `rate()` should only be used with Counter metrics
- Alert states: `inactive` → `pending` → `firing` → `resolved`