# Monitoring Stack with Prometheus, Alertmanager, Grafana, and Exporters

# 📖 Introduction

This project sets up a **containerized monitoring and alerting system** using **Prometheus**, **Grafana**, **Alertmanager**, **Node Exporter**, and **Blackbox Exporter** using Docker Compose. It enables system and application health tracking along with email notifications for critical alerts.

---

# 📦 Prerequisites

Before you begin, ensure you have the following installed:

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- SMTP-enabled email account (e.g., Gmail)
  - App Password is required for Gmail (NOT your main password).

---

# 📁 Project Structure

```bash
monitoring-project/
├── alerts/
│   └── alertmanager.yml
├── grafana/
│   └── grafana.ini
├── prometheus/
│   └── prometheus.yml
├── rules/
│   └── alert-rules.yml
├── docker-compose.yml
```

---

# 🚀 Getting Started

### 1. Clone the Repository or Create the Folder Structure

```bash
mkdir monitoring-project && cd monitoring-project
```

Create the files and folders as shown in the structure above.

### 2. Launch All Services

```bash
docker-compose up -d
```

This will start:
- Prometheus (port `9090`)
- Node Exporter (port `9100`)
- Blackbox Exporter (port `9115`)
- Alertmanager (port `9093`)
- Grafana (port `3000`)

---

# ⚙️ Service Details

## ✅ Node Exporter - `localhost:9100`

**Purpose:** Exposes hardware and OS-level metrics.

### 📦 Docker Compose Configuration:
```yaml
node-exporter:
  container_name: node-exporter
  image: prom/node-exporter
  ports:
    - 9100:9100
```

### 🔍 Metrics Exposed:
- CPU Usage
- Memory Usage
- Disk I/O
- Filesystem space

### 🔧 Prometheus Scrape Configuration:
```yaml
- job_name: 'node_exporter'
  static_configs:
    - targets: ['node-exporter:9100']
```

This tells Prometheus to pull metrics from the `node-exporter` container on port `9100` every 15s.

---

## ✅ Blackbox Exporter - `localhost:9115`

**Purpose:** Actively probes endpoints (HTTP, TCP, ICMP) and reports availability.

### 📦 Docker Compose Configuration:
```yaml
blackbox_exporter:
  image: prom/blackbox-exporter:v0.25.0
  container_name: blackbox_exporter
  ports:
    - "9115:9115"
```

### 🔧 Prometheus Scrape Configuration:
```yaml
- job_name: 'blackbox'
  metrics_path: /probe
  params:
    module: [http_2xx]
  static_configs:
    - targets:
        - http://localhost:9115
  relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: instance
    - target_label: __address__
      replacement: blackbox_exporter:9115
```

### 💡 Explanation:
- `metrics_path`: Custom endpoint `/probe` is used for probing.
- `module`: `http_2xx` is the default probe for HTTP status 200 checks.
- `relabel_configs`: Transforms target into parameters for Blackbox to probe properly.

---

## ✅ Alertmanager - `localhost:9093`

**Purpose:** Receives alerts from Prometheus and sends notifications (e.g., email).

### 📦 Docker Compose Configuration:
```yaml
alert-manager:
  container_name: alert-manager
  image: prom/alertmanager
  ports:
    - 9093:9093
  volumes:
    - ./alerts/alertmanager.yml:/etc/alertmanager/alertmanager.yml
```

### 🔧 Prometheus Alerting Configuration:
```yaml
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alert-manager:9093']
```

### 📂 alertmanager.yml Configuration:
```yaml
global:
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: 'your_gmail@gmail.com'
  smtp_auth_username: 'your_gmail@gmail.com'
  smtp_auth_password: 'your_app_password'
  smtp_require_tls: true

route:
  receiver: 'email-alert'

receivers:
  - name: 'email-alert'
    email_configs:
      - to: 'your_gmail@gmail.com'
```

### 💡 Explanation:
- Uses Gmail SMTP with app password for sending emails.
- Routes all alerts to `email-alert` receiver.

---

# 📊 Grafana Configuration

### 📦 Docker Compose:
```yaml
grafana:
  image: grafana/grafana
  container_name: grafana
  ports:
    - 3000:3000
  volumes:
    - grafana-storage:/var/lib/grafana
    - ./grafana/grafana.ini:/etc/grafana/grafana.ini
  depends_on:
    - prometheus
```

### 📂 grafana.ini:
```ini
[server]
http_port = 3000

[security]
admin_user = admin
admin_password = admin

[smtp]
enabled = true
host = smtp.gmail.com:587
user = your_gmail@gmail.com
password = your_app_password
from_address = your_gmail@gmail.com
from_name = Grafana Alerts
skip_verify = false
startTLS_policy = opportunistic
```

### 📥 Accessing Grafana
- URL: [http://localhost:3000](http://localhost:3000)
- Login: `admin / admin`
- Add Prometheus as a data source using `http://prometheus:9090`

---

# 📧 Email Alerting Setup

- Email alerts are configured through both **Alertmanager** and **Grafana**.
- **App password** is used instead of the actual Gmail password.
- Alerts will be triggered for conditions defined in `alert-rules.yml`.

---

# 🔒 Security Notes

- Never commit sensitive files (like SMTP passwords) to public repositories.
- Restrict firewall access to ports `3000`, `9090`, `9100`, `9093`, and `9115`.
- Use `.env` files or secrets management for passwords in production.

---

# ✅ Verification

1. Visit Prometheus UI → [http://localhost:9090](http://localhost:9090)
   - Confirm targets are UP in **Status > Targets**
2. Check Node Exporter metrics → [http://localhost:9100/metrics](http://localhost:9100/metrics)
3. Visit Grafana UI → [http://localhost:3000](http://localhost:3000)
   - Add Prometheus as data source and create dashboards
4. Visit Alertmanager → [http://localhost:9093](http://localhost:9093)
   - Confirm alerts and routes

---

# 🧼 Cleanup

To stop and remove containers:
```bash
docker-compose down
```

To remove all data:
```bash
docker system prune -a --volumes
```

---

This concludes the detailed documentation. You now have a fully functioning monitoring and alerting system using Prometheus, Node Exporter, Blackbox Exporter, Alertmanager, and Grafana. 🎉

