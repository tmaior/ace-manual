# ACE Monitoring Stack (Prometheus/Grafana/Alertmanager + Loki)

This page explains where the ACE monitoring stack is documented and the minimum operational steps to deploy, access, and validate it.

The authoritative docs live in `ace-infra/monitoring/`.

---

## Components

- Prometheus: metrics collection and storage
- Grafana: dashboards
- Alertmanager: alert routing and notifications
- Loki: log aggregation
- Promtail: log shipping to Loki

---

## Where the docs and scripts are

In the `ace-infra` repository:

- Deploy: `ace-infra/monitoring/deploy.sh`
- Access info: `ace-infra/monitoring/info.sh`
- Port-forward helper: `ace-infra/monitoring/port-forward.sh`
- Uninstall: `ace-infra/monitoring/uninstall.sh`
- Main guides:
  - `ace-infra/monitoring/START-HERE.md`
  - `ace-infra/monitoring/README.md`
  - `ace-infra/monitoring/QUICK-START.md`
  - `ace-infra/monitoring/EXAMPLE-ADD-METRICS.md`
  - `ace-infra/monitoring/ALERTING-GUIDE.md`

---

## Kubernetes namespace

The stack is deployed into the `monitoring` namespace.

---

## Quick deploy (from docs)

```bash
cd /home/admin/ace/ace-infra/monitoring
./deploy.sh
```

---

## Access Grafana

Primary:

```bash
cd /home/admin/ace/ace-infra/monitoring
./info.sh
```

Alternative (port-forward):

```bash
kubectl port-forward -n monitoring svc/prometheus-stack-grafana 3000:80
# http://localhost:3000
```

---

## How services get monitored

ACE monitoring relies on Kubernetes `ServiceMonitors` (Prometheus Operator) and service labels.

Minimum requirement (per ace-infra docs):

- `app.kubernetes.io/part-of: ace`

The exact monitored service list and dashboard IDs are defined in `ace-infra/monitoring/README.md` and linked from `ace-infra/monitoring/INDEX.md`.

---

## Alerts

Alert rules are applied as Kubernetes `PrometheusRule` resources. See:

- `ace-infra/monitoring/ALERTING-GUIDE.md`
- `ace-infra/monitoring/k8s/alerting-rules-example.yaml` (example rules)
- `ace-infra/monitoring/helm/values-prometheus.yaml` (Alertmanager configuration)

