# Prometheus (PCA) Sample Questions Lab

A hands-on Prometheus lab demonstrating key concepts covered in the **Prometheus (PCA) Sample Questions** video.

©️ Dave Prowse

📺 **Watch the video:** [INSERT YOUTUBE LINK HERE]

📄 **Free Prometheus (PCA) Cram Sheet & Sample Questions:** https://prowse-tech.kit.com/prometheus-cramsheet

🎓 **Full Prometheus (PCA) Video Course (free 10-day trial):** https://learning.oreilly.com/course/prometheus-certified-associate/9780135556139/

---

## What This Lab Covers

- Binary installation of Prometheus, Node Exporter, and Alertmanager
- Configuring Prometheus to scrape a Node Exporter target
- Basic and intermediate PromQL queries in the Web UI and with curl
- Configuring Alertmanager and writing alert rules
- Observing alert state transitions: `inactive` → `pending` → `firing` → `resolved`

## Requirements

- Two Debian-based Linux VMs (VirtualBox or similar)
  - **VM1:** Prometheus server + Alertmanager
  - **VM2:** Node Exporter (monitored target)
- Internet access to download binaries from https://prometheus.io/download/
- No cloud credentials required

> **Note:** Node Exporter can also run on VM1 for a simpler single-VM setup.

## Usage

```bash
git clone https://github.com/daveprowse/prometheus-sample-questions
cd prometheus-sample-questions
```

Follow the steps in [LAB.md](LAB.md) in order.

## Lab Structure

```
├── README.md           # This file
├── LAB.md              # Step-by-step lab instructions
├── prometheus.yml      # Prometheus configuration
└── alert_rules.yml     # Alerting rules
```

## Key Ports

| Component | Default Port |
|-----------|-------------|
| Prometheus | 9090 |
| Node Exporter | 9100 |
| Alertmanager | 9093 |

---

## About

Created by **Dave Prowse** — author of the best-selling A+ Exam Cram and Prometheus (PCA) Video Course on O'Reilly.

- 🌐 Website: https://prowse.tech
- 🎥 YouTube: https://youtube.com/@prowsetech
- 💬 Discord: https://discord.com/invite/mggw8VGzUp