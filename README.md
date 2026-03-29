# End-to-End Observability Stack on Kubernetes
 
## Overview
Complete observability implementation for a Kubernetes-based microservices platform.
Covers metrics collection, centralized logging, and distributed tracing.
 
## Observability Architecture
 
  [App Pods]
     |-- metrics (scrape every 15s) -----------> [Prometheus] --> [Grafana]
     |-- stdout logs (DaemonSet) -------------> [Fluentd] --> [Elasticsearch] --> [Kibana]
     |-- traces (OTLP gRPC port 4317) ---------> [Jaeger Collector] --> [Jaeger UI]
 
  [Grafana] <-- Jaeger data source (trace lookup by ID)
  [Kibana]  <-- search by trace_id from structured logs
 
## Component Reference
| Component     | Namespace  | Port  | Purpose                         |
|---------------|------------|-------|---------------------------------|
| Prometheus    | monitoring | 9090  | Metrics scraping and storage    |
| Grafana       | monitoring | 3000  | Metrics visualization           |
| Elasticsearch | logging    | 9200  | Log indexing and storage        |
| Fluentd       | logging    |   -   | Log collection (DaemonSet)      |
| Kibana        | logging    | 5601  | Log search and visualization    |
| Jaeger        | tracing    | 16686 | Distributed trace UI            |
 
## How Components Integrate
1. METRICS: Prometheus discovers app pods via kubernetes_sd_configs and
   scrapes pods annotated with prometheus.io/scrape=true every 15 seconds.
   Grafana queries Prometheus via PromQL to render real-time dashboards.
 
2. LOGS: Fluentd runs as a DaemonSet, tails /var/log/containers on every
   node, enriches logs with K8s metadata (namespace, pod, container),
   and forwards to Elasticsearch. Kibana provides full-text search.
 
3. TRACES: Apps use the OpenTelemetry SDK. Spans are exported via OTLP
   to Jaeger Collector running in the tracing namespace. Jaeger UI shows
   flame graphs of multi-service request flows.
 
4. CORRELATION: Apps embed trace_id in every structured JSON log line.
   Search Kibana by trace_id to find all logs for a request.
   Open Jaeger with the same ID to see the full trace flame graph.
   Grafana links panels to Jaeger via the Jaeger data source.
 
## Repository Structure
  manifests/
    namespaces.yaml
    prometheus-config.yaml
    prometheus-alerts.yaml
    grafana-app-dashboard.yaml
