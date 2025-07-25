# Kubernetes Monitoring Stack Demo

A comprehensive monitoring solution using Prometheus, Grafana, and AlertManager to provide observability for Kubernetes clusters and applications.

## 📋 Overview

This project demonstrates a complete monitoring stack deployment that provides metrics collection, visualization, and alerting for Kubernetes environments. It includes pre-configured dashboards, alert rules, and best practices for production monitoring.

## 🛠️ Technologies Used

- **Metrics Collection:** Prometheus
- **Visualization:** Grafana
- **Alerting:** AlertManager
- **Service Discovery:** Kubernetes API
- **Exporters:** Node Exporter, cAdvisor
- **Container Runtime:** Docker Compose / Kubernetes

---

## 📂 Project Structure

```
k8s-monitoring-stack-demo/
├── docker-compose.yml      # Local development setup
├── kubernetes/             # Kubernetes manifests
│   ├── namespace.yaml
│   ├── prometheus/
│   ├── grafana/
│   └── alertmanager/
├── configs/                # Configuration files
│   ├── prometheus.yml
│   ├── alert-rules.yml
│   └── grafana-datasources.yml
├── dashboards/             # Grafana dashboard JSONs
│   ├── cluster-overview.json
│   ├── node-metrics.json
│   └── application-metrics.json
├── scripts/                # Utility scripts
│   ├── setup.sh
│   └── cleanup.sh
└── README.md
```

---

## 🔍 Monitoring Components

### Prometheus Stack
- **Prometheus Server:** Metrics collection and storage
- **Node Exporter:** System and hardware metrics
- **cAdvisor:** Container resource metrics
- **Kube State Metrics:** Kubernetes object metrics
- **Custom Exporters:** Application-specific metrics

### Grafana Stack
- **Grafana Server:** Metrics visualization platform
- **Pre-built Dashboards:** Ready-to-use monitoring views
- **Data Sources:** Prometheus integration
- **User Management:** Role-based access control

### AlertManager
- **Alert Rules:** Proactive monitoring alerts
- **Notification Channels:** Slack, Email, Webhook
- **Alert Grouping:** Intelligent alert management
- **Silence Rules:** Alert suppression during maintenance

---

## 🚀 Quick Start

### Docker Compose (Local Development)

1. **Clone the repository:**
   ```bash
   git clone https://github.com/hamelin123/k8s-monitoring-stack-demo.git
   cd k8s-monitoring-stack-demo
   ```

2. **Start the monitoring stack:**
   ```bash
   docker-compose up -d
   ```

3. **Access the services:**
   - **Prometheus:** http://localhost:9090
   - **Grafana:** http://localhost:3000 (admin/admin)
   - **AlertManager:** http://localhost:9093

### Kubernetes Deployment

1. **Create monitoring namespace:**
   ```bash
   kubectl apply -f kubernetes/namespace.yaml
   ```

2. **Deploy Prometheus:**
   ```bash
   kubectl apply -f kubernetes/prometheus/
   ```

3. **Deploy Grafana:**
   ```bash
   kubectl apply -f kubernetes/grafana/
   ```

4. **Deploy AlertManager:**
   ```bash
   kubectl apply -f kubernetes/alertmanager/
   ```

---

## 📊 Monitoring Dashboards

### Cluster Overview Dashboard
- **CPU Usage:** Cluster-wide CPU utilization
- **Memory Usage:** Memory consumption across nodes
- **Pod Status:** Running, pending, failed pods
- **Network I/O:** Cluster network traffic
- **Storage:** Persistent volume usage

### Node Metrics Dashboard
- **System Load:** CPU load averages
- **Memory Details:** RAM usage, swap, buffers
- **Disk I/O:** Read/write operations and latency
- **Network Details:** Interface statistics
- **Temperature:** Hardware temperature monitoring

### Application Metrics Dashboard
- **Request Rate:** HTTP requests per second
- **Response Time:** Application latency percentiles
- **Error Rate:** 4xx/5xx error tracking
- **Database Metrics:** Query performance and connections
- **Custom Metrics:** Application-specific KPIs

---

## ⚙️ Configuration

### Prometheus Configuration
```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "alert-rules.yml"

scrape_configs:
  - job_name: 'kubernetes-nodes'
    kubernetes_sd_configs:
    - role: node
    relabel_configs:
    - source_labels: [__address__]
      regex: '(.*):10250'
      target_label: __address__
      replacement: '${1}:9100'
```

### Alert Rules Example
```yaml
# alert-rules.yml
groups:
- name: kubernetes-alerts
  rules:
  - alert: NodeDown
    expr: up{job="kubernetes-nodes"} == 0
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Node {{ $labels.instance }} is down"
      description: "Node has been down for more than 5 minutes"
```

### Grafana Dashboard Configuration
```json
{
  "dashboard": {
    "title": "Kubernetes Cluster Overview",
    "panels": [
      {
        "title": "CPU Usage",
        "type": "graph",
        "targets": [
          {
            "expr": "100 - (avg(irate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)"
          }
        ]
      }
    ]
  }
}
```

---

## 📈 Metrics and KPIs

### System Metrics
- **CPU Utilization:** Current and historical usage
- **Memory Usage:** RAM, swap, and cache metrics
- **Disk I/O:** Read/write IOPS and throughput
- **Network Traffic:** Bytes in/out, packets, errors
- **Load Average:** System load over time

### Kubernetes Metrics
- **Pod Metrics:** CPU, memory, restart count
- **Deployment Status:** Replicas, availability
- **Service Metrics:** Endpoint availability
- **Ingress Metrics:** Request rate, response codes
- **Persistent Volume:** Usage and availability

### Application Metrics
- **HTTP Metrics:** Request rate, latency, errors
- **Database Metrics:** Connection pool, query time
- **Cache Metrics:** Hit rate, evictions
- **Queue Metrics:** Message throughput, lag
- **Custom Business Metrics:** User-defined KPIs

---

## 🔔 Alerting Configuration

### Critical Alerts
- **Node Down:** Node becomes unresponsive
- **High CPU Usage:** CPU usage > 90% for 5 minutes
- **Memory Pressure:** Memory usage > 85%
- **Disk Space:** Disk usage > 90%
- **Pod Restart Loop:** Pod restarts > 5 times in 10 minutes

### Warning Alerts
- **High Load:** Load average > 80%
- **Slow Response Time:** API latency > 1 second
- **Error Rate:** Error rate > 5%
- **Certificate Expiry:** SSL certificates expiring in 30 days

### Notification Channels
```yaml
# alertmanager.yml
route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'web.hook'

receivers:
- name: 'web.hook'
  slack_configs:
  - api_url: 'YOUR_SLACK_WEBHOOK_URL'
    channel: '#alerts'
    title: 'Kubernetes Alert'
    text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
```

---

## 🎯 Key Features

- **Auto Discovery:** Automatic service discovery
- **Multi-Cluster:** Support for multiple clusters
- **Custom Metrics:** Easy custom metric integration
- **High Availability:** Redundant monitoring setup
- **Data Retention:** Configurable metric retention
- **Security:** RBAC and TLS enabled

---

## 🛠️ Maintenance

### Backup and Restore
```bash
# Backup Grafana dashboards
kubectl get configmap grafana-dashboards -o yaml > dashboards-backup.yaml

# Backup Prometheus data
kubectl exec prometheus-0 -- tar czf - /prometheus | gzip > prometheus-backup.tar.gz
```

### Scaling
```bash
# Scale Prometheus replicas
kubectl scale statefulset prometheus --replicas=2

# Update resource limits
kubectl patch deployment grafana -p '{"spec":{"template":{"spec":{"containers":[{"name":"grafana","resources":{"limits":{"memory":"512Mi"}}}]}}}}'
```

### Health Checks
```bash
# Check Prometheus health
curl http://localhost:9090/-/healthy

# Check Grafana health  
curl http://localhost:3000/api/health

# Check AlertManager health
curl http://localhost:9093/-/healthy
```

---

## 📚 Documentation

- [Prometheus Configuration Guide](docs/prometheus-setup.md)
- [Grafana Dashboard Creation](docs/grafana-dashboards.md)
- [Alert Rules Configuration](docs/alerting.md)
- [Troubleshooting Guide](docs/troubleshooting.md)

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Test with local Docker Compose
4. Submit a pull request

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

<div align="center">
  <p>📊 Observability Made Simple</p>
  <p>Built with 💚 for DevOps Engineers</p>
</div>
