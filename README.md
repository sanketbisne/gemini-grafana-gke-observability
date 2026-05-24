# AI-Powered Observability on GKE: Query Grafana with Gemini CLI & Gemini Extensions

[![Kubernetes](https://img.shields.io/badge/Kubernetes-GKE-blue.svg?logo=kubernetes&style=flat-square)](#)
[![Grafana](https://img.shields.io/badge/Grafana-10.0.0-orange.svg?logo=grafana&style=flat-square)](#)
[![Prometheus](https://img.shields.io/badge/Prometheus-v2.45.0-red.svg?logo=prometheus&style=flat-square)](#)
[![Gemini](https://img.shields.io/badge/Gemini-Extensions--MCP-purple.svg?logo=google-gemini&style=flat-square)](#)

This repository contains the complete demo architecture, manifests, and live scripts for the **GrafanaCon** session: 
**"AI-Powered Observability on GKE: Query Grafana with Gemini CLI & Gemini Extensions"**.

This session demonstrates how modern SREs and DevOps engineers can build an AI-driven, conversational observability platform. By deploying Prometheus and Grafana on Google Kubernetes Engine (GKE) and connecting them to **Gemini CLI** via the **Model Context Protocol (MCP)**, you can troubleshoot incidents, query metrics, and manage dashboards using pure natural language.

---

## 🏗️ Architecture Overview

The demo environment consists of a fully-provisioned observability sandbox running inside GKE:

```mermaid
graph TD
    subgraph GKE Cluster (Google Kubernetes Engine)
        subgraph Applications
            SA[Sample Nginx App]
            NXE[Nginx Prometheus Exporter]
        end
        
        subgraph Monitoring
            PROM[Prometheus Server]
            GRAF[Grafana Dashboard]
        end
        
        SA -- Exposes stub_status --> NXE
        PROM -- Scrapes Metrics --> NXE
        PROM -- Scrapes Node & Pod cAdvisor --> PROM
        GRAF -- Queries Metrics --> PROM
    end

    subgraph Developer Laptop
        CLI[Gemini CLI]
        MCP[mcp-grafana extension]
    end

    CLI <--> MCP
    MCP -- API calls via HTTP (Port-forward/LoadBalancer) --> GRAF
```

### Sandbox Components
1. **Sample App ([sample-app.yaml](file:///Users/sanketbisne/k8s/sample-app.yaml))**: An Nginx container with a sidecar Nginx Prometheus Exporter that exposes server-level metrics (`/nginx_status`).
2. **Prometheus ([prometheus.yaml](file:///Users/sanketbisne/k8s/prometheus.yaml))**: Automatically scrapes cluster-wide cAdvisor metrics (CPU, Memory) as well as any pods marked with Prometheus annotations.
3. **Grafana ([grafana.yaml](file:///Users/sanketbisne/k8s/grafana.yaml))**: Pre-configured with a Prometheus datasource and a pre-loaded **Kubernetes Resource Usage** dashboard showcasing pod-level CPU and memory graphs.

---

## 🚀 Quickstart: Deployment on GKE

Follow these steps to deploy the entire stack to your Google Kubernetes Engine (GKE) cluster.

### Prerequisites
* A running **GKE Cluster**
* **kubectl** configured to access your cluster
* **Gemini CLI** installed on your workstation
* Grafana access token with Admin privileges

---

### Step 1: Deploy Prometheus Server
Prometheus acts as the time-series database. It is configured with ClusterRole permissions to discover nodes, pods, and services dynamically.

```bash
kubectl apply -f prometheus.yaml
```

*This deploys:*
* ServiceAccount, ClusterRole, and ClusterRoleBinding for Kubernetes service discovery.
* A ConfigMap containing the scrape configurations.
* A single-replica deployment running Prometheus `v2.45.0`.
* A cluster IP service mapping port `9090`.

---

### Step 2: Deploy Grafana
Grafana is pre-provisioned with the Prometheus data source and an interactive Kubernetes resource usage dashboard.

```bash
kubectl apply -f grafana.yaml
```

*This deploys:*
* Grafana ConfigMaps for automatic provisioning of the data source and the Kubernetes dashboard.
* A deployment running Grafana `10.0.0` with anonymous Admin access enabled (ideal for demos).
* A `LoadBalancer` service exposing Grafana externally on port `80`.

---

### Step 3: Deploy the Sample Nginx Application
A workload to generate telemetry data and test scraping capabilities.

```bash
kubectl apply -f sample-app.yaml
```

*This deploys:*
* An Nginx deployment with a ConfigMap setting up `nginx.conf` and exposing `/nginx_status`.
* A sidecar container running `nginx-prometheus-exporter` on port `9113`.
* A standard Kubernetes service targeting port `80`.

---

### Step 4: Verify Deployments
Ensure all pods are in a `Running` state:

```bash
kubectl get pods
```

Retrieve the external IP of the Grafana service to access the dashboard and use it for Gemini CLI setup:

```bash
kubectl get svc grafana -w
```

---

## 🤖 Integrating Gemini CLI with Grafana (MCP)

To enable conversational observability, you will install the official Grafana Model Context Protocol (MCP) server directly inside the Gemini CLI. This allows Gemini to invoke APIs under the hood when you ask questions about dashboards, panels, and metrics.

### 🔌 Installation
Run the following installation command in your terminal where Gemini CLI is installed:

```bash
gemini extensions install https://github.com/grafana/mcp-grafana
```

---

## 🔍 Inspecting MCP Extension Capabilities

Once the extension is installed, you can query and inspect the active tools in your Gemini session.

### Running `/mcp list` or `mcp list`
Typing `mcp list` in the Gemini CLI will display all connected MCP Servers and their respective toolsets. When configured with Grafana, you will see a dynamic output similar to:

```text
✦ I have access to several Model Context Protocol (MCP) servers that extend my capabilities. Here is a summary of the available toolsets:

  1. Google Cloud Run (mcp_cloud-run)
     Allows for management and deployment of services on Google Cloud.
  
  2. GitHub (mcp_github)
     Comprehensive repository management and collaboration.

  3. Grafana (mcp_grafana)
     Observability and incident management tools:
      * Dashboards & Panels: Search, retrieve, and update dashboards; render panels as images.
      * Alerting: Manage alert rules and notification routing.
      * OnCall & Incidents: Manage incidents, schedules, shifts, and on-call users.
      * Data Queries: Query Prometheus (PromQL), Loki (LogQL), Pyroscope, and Tempo.
      * Sift: Perform automated investigations for error patterns and slow requests.
```

If you query details about active tools using `/mcp list`, it reveals the **52 high-fidelity tools** available on the Grafana server:

```text
🟢 grafana (from grafana) - Ready (52 tools)
  Tools:
  - mcp_grafana_add_activity_to_incident
  - mcp_grafana_alerting_manage_routing
  - mcp_grafana_alerting_manage_rules
  - mcp_grafana_create_annotation
  - mcp_grafana_create_folder
  - mcp_grafana_create_incident
  - mcp_grafana_find_error_pattern_logs
  - mcp_grafana_find_slow_requests
  - mcp_grafana_generate_deeplink
  - mcp_grafana_get_alert_group
  - mcp_grafana_get_annotation_tags
  - mcp_grafana_get_annotations
  - mcp_grafana_get_assertions
  - mcp_grafana_get_current_oncall_users
  - mcp_grafana_get_dashboard_by_uid
  - mcp_grafana_get_dashboard_panel_queries
  - mcp_grafana_get_dashboard_property
  - mcp_grafana_get_dashboard_summary
  - mcp_grafana_get_datasource
  - mcp_grafana_get_incident
  - mcp_grafana_get_oncall_shift
  - mcp_grafana_get_panel_image
  - mcp_grafana_get_plugin
  - mcp_grafana_get_sift_analysis
  - mcp_grafana_get_sift_investigation
  - mcp_grafana_grafana_api_request
  - mcp_grafana_list_alert_groups
  - mcp_grafana_list_datasources
  - mcp_grafana_list_incidents
  - mcp_grafana_list_loki_label_names
  - mcp_grafana_list_loki_label_values
  - mcp_grafana_list_oncall_schedules
  - mcp_grafana_list_oncall_teams
  - mcp_grafana_list_oncall_users
  - mcp_grafana_list_prometheus_label_names
  - mcp_grafana_list_prometheus_label_values
  - mcp_grafana_list_prometheus_metric_metadata
  - mcp_grafana_list_prometheus_metric_names
  - mcp_grafana_list_pyroscope_label_names
  - mcp_grafana_list_pyroscope_label_values
  - mcp_grafana_list_pyroscope_profile_types
  - mcp_grafana_list_sift_investigations
  - mcp_grafana_query_loki_logs
  - mcp_grafana_query_loki_patterns
  - mcp_grafana_query_loki_stats
  - mcp_grafana_query_prometheus
  - mcp_grafana_query_prometheus_histogram
  - mcp_grafana_query_pyroscope
  - mcp_grafana_search_dashboards
  - mcp_grafana_search_folders
  - mcp_grafana_update_annotation
  - mcp_grafana_update_dashboard
```

---

## 🛡️ SRE Security: The Interactive Consent UI

One of the key security pillars in Gemini CLI is its **explicit human-in-the-loop consent mechanism**. Gemini will never run a destructive or analytical action against your live systems without asking.

When you ask Gemini CLI to run an observation command:
> *"List my Grafana dashboards"*

The CLI intercepts the matching MCP tool (`list_datasources` or `search_dashboards`) and presents a visual consent modal in the terminal:

```text
╭─────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Action Required                                                                                                 │
│                                                                                                                 │
│ ?  list_datasources (grafana MCP Server) {}                                                                     │
│                                                                                                                 │
│ MCP Server: grafana                                                                                             │
│ Tool: list_datasources                                                                                          │
│                                                                                                                 │
│ Allow execution of MCP tool "list_datasources" from server "grafana"?                                           │
│                                                                                                                 │
│ ● 1. Allow once                                                                                                 │
│   2. Allow tool for this session                                                                                │
│   3. Allow all server tools for this session                                                                    │
│   4. No, suggest changes (esc)                                                                                  │
╰─────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
```

This security mechanism gives operators full control over the execution scope: from single-time execution (`Allow once`) to full-session automation (`Allow all server tools for this session`).

---

## 🧠 SRE Agent Intelligence: Local Workspace Awareness & Fallback

A standout feature of Gemini CLI is **contextual local workspace awareness**. If the live Grafana connection encounters issues (e.g. invalid credentials, networking problems, or a `401 Unauthorized` token error), the agent does not just crash or show a blank screen.

### Real-World Demo Scenario: Gracious API Token Fallback
If the live connection fails:

```text
  Listing Dashboards: Searching for all Grafana dashboards to provide a list to the user.

  ✓  ReadFile  grafana.yaml

✦ The attempt to list dashboards from your Grafana instance failed with a 401 Unauthorized error (invalid API key). 
  This indicates that the Grafana MCP server is not correctly authenticated with your Grafana instance.

  However, I found a dashboard definition in your local grafana.yaml file:

  Kubernetes Resource Usage (uid: k8s-resources)
  This dashboard is pre-provisioned via a ConfigMap and includes the following panels:
   * CPU Usage by Pod: sum(rate(container_cpu_usage_seconds_total{container!=\"\"}[5m])) by (pod)
   * Memory Usage by Pod: sum(container_memory_working_set_bytes{container!=\"\"}) by (pod)
```

**Why this is a killer talking point at GrafanaCon:**
* **Local Parsing**: The agent autonomously notices `grafana.yaml` in the user's open files or active workspace directory.
* **Intelligent Synthesis**: It parses the ConfigMap inside `grafana.yaml`, extracts the JSON definition of the dashboard (`Kubernetes Resource Usage` with UID `k8s-resources`), identifies the metric panels, and serves the user with the PromQL expressions directly.
* **Hybrid Operation**: This demonstrates how Gemini blends live runtime connections with local code context for an uninterrupted SRE workspace experience.

---

## 🎭 Live Demo Walkthrough: Conversational Observability

Use these highly interactive scenarios during the GrafanaCon presentation to showcase real-world workflows.

### Scenario A: Dashboard Discovery & Inspection
Instead of manually navigating the UI, ask Gemini to inspect your Grafana instance.

| SRE Natural Language Query | What Gemini Does Behind the Scenes (MCP Tools) |
|---|---|
| *"What dashboards are currently configured in my Grafana instance?"* | Invokes `mcp_grafana_search_dashboards` to fetch the metadata of all loaded boards. |
| *"Describe the panels inside the 'Kubernetes Resource Usage' dashboard."* | Invokes `mcp_grafana_get_dashboard_by_uid` with UID `k8s-resources` to outline the panels ("CPU Usage by Pod", "Memory Usage by Pod"). |
| *"What are the Prometheus data sources available?"* | Invokes `mcp_grafana_list_datasources` to confirm connection configurations. |

---

### Scenario B: Diagnostic Reasoning & Metrics Scraper Check
Let Gemini analyze your actual Kubernetes targets:

1. **Ask Gemini for a high-level status of CPU utilization:**
   > *"Sanket: Run a PromQL query on my datasource to check the CPU usage of our pods."*
2. **Dynamic Querying:**
   > *(Gemini CLI invokes `mcp_grafana_query_prometheus` using the expression parsed from your local configuration: `sum(rate(container_cpu_usage_seconds_total{container!=\"\"}[5m])) by (pod)` and reports the live utilization metrics in clean markdown tables.)*

---

## 🛠️ Customization & Extensibility

* **Adding custom metrics**: You can modify the metrics dashboard in [grafana.yaml](file:///Users/sanketbisne/k8s/grafana.yaml) under the ConfigMap `grafana-dashboard-k8s`.
* **Updating Prometheus scrape jobs**: Edit [prometheus.yaml](file:///Users/sanketbisne/k8s/prometheus.yaml) under the `prometheus-server-conf` ConfigMap to add custom rules or scrape jobs for additional microservices.

---

> [!TIP]
> **For the Live Demo at GrafanaCon:** Make sure to port-forward Prometheus (`kubectl port-forward svc/prometheus-service 9090:9090`) and Grafana (`kubectl port-forward svc/grafana 3000:80`) if you are running on a local cluster like Minikube, or use the `LoadBalancer` external IP on GKE for seamless external access.
