 # 📊 Prometheus In Action: Multi-Node & Flask Application Monitoring
 
 A complete, production-ready demonstration package illustrating how to monitor a distributed environment and instrument a custom Python Flask application with Prometheus metrics using Docker containers.

---

## 🛠️ Tech Stack

<p align="left">
  <a href="https://skillicons.dev">
    <img src="https://skillicons.dev/icons?i=prometheus,docker,python,flask" alt="Tech Stack Icons" />
  </a>
</p>

- **Prometheus**: Time-series database and monitoring/alerting server.
- **Docker & Docker Compose**: Containerization and network isolation.
- **Python 3.9 & Flask**: Custom microservice implementation.
- **prometheus-flask-exporter**: Native Python metrics instrumentation library.
- **Node Exporter**: Hardware and OS metrics exporter.

---

## 📁 Repository Structure

```text
.
├── Dockerfile             # Container configuration for the Python Flask application
├── README.md              # Project documentation & execution guide
├── prometheus.yml         # Prometheus scraping and job target configuration
├── pythonserver.py        # Instrumented Python Flask server
└── images/                # Screenshots illustrating system behavior (Placeholders)
    ├── targets-up.png     # Prometheus target status page with all nodes UP
    ├── node1-down.png     # Prometheus target status showing node-exporter1 DOWN
    └── app-metrics.png    # Graph showing Flask HTTP request metrics & traffic
```

---

## 🚀 Quick Start & Usage

This project deploys a virtual environment comprising 3 Node Exporters, a custom Python Flask app, and a central Prometheus instance inside a Docker bridge network named `monitor`.

### 1. Set Up the Network & Node Exporters
Run the following commands to create the network isolation layer and launch three Node Exporter containers to simulate distinct infrastructure nodes:

```bash
# Create the isolated bridge network
docker network create monitor

# Start Node Exporter 1 (Mapped to port 9101)
docker run -d --name node-exporter1 -p 9101:9100 --network monitor bitnami/node-exporter:latest

# Start Node Exporter 2 (Mapped to port 9102)
docker run -d --name node-exporter2 -p 9102:9100 --network monitor bitnami/node-exporter:latest

# Start Node Exporter 3 (Mapped to port 9103)
docker run -d --name node-exporter3 -p 9103:9100 --network monitor bitnami/node-exporter:latest
```

Verify that the exporter nodes are up and running:
```bash
docker ps | grep node-exporter
```

---

### 2. Build & Deploy the Custom Flask Application
The Python application `pythonserver.py` is pre-instrumented using the `prometheus-flask-exporter` library. Build and run it using the following commands:

```bash
# Build the application image
docker build -t pythonserver .

# Run the container (Exposed to port 8081)
docker run -d --name pythonserver -p 8081:8080 --network monitor pythonserver
```

---

### 3. Deploy Prometheus
Start the Prometheus container, mounting the local `prometheus.yml` configuration:

```bash
docker run -d --name prometheus -p 9090:9090 --network monitor \
  -v $(pwd)/prometheus.yml:/opt/bitnami/prometheus/conf/prometheus.yml \
  bitnami/prometheus:latest
```

---

## 📈 Monitoring & Verification

### Target Status Check
Navigate to the Prometheus Web UI at `http://localhost:9090/targets` to verify all endpoints are correctly registered and scraped:

![Prometheus Targets UI showing nodes up](./images/targets-up.png)

### Simulated Outage Response
To test alerting and observability capability, shut down the first node exporter:
```bash
docker stop node-exporter1
```
Observe the state transition within the Prometheus Target dashboard after the scraping interval (15s):

![Node Exporter 1 Down](./images/node1-down.png)

### Metric Scraping & Traffic Verification
Generate sample traffic on the instrumented Flask microservice:
```bash
curl localhost:8081/
curl localhost:8081/home
curl localhost:8081/contact
```

Run Prometheus queries (e.g., `flask_http_request_total` or `flask_http_request_duration_seconds_bucket`) in the Graph page to monitor traffic patterns:

![Prometheus Graph showing HTTP requests](./images/app-metrics.png)

---

## 📷 Screenshot Asset Requirements

To complete the visual walkthrough of this portfolio piece, capture and save the following screenshots from your active lab environment to the `./images/` directory:

*   **`targets-up.png`**: The Prometheus Targets dashboard (`http://localhost:9090/targets`) with all configured endpoints (`node-exporter1`, `node-exporter2`, `node-exporter3`, and `pythonserver`) in the green **UP** status.
*   **`node1-down.png`**: The Targets dashboard showing `node-exporter1:9100` in the red **DOWN** status after performing the simulated outage task.
*   **`app-metrics.png`**: The Prometheus Graph console tracking metrics such as `flask_http_request_total` or `flask_http_request_duration_seconds_bucket` to show active requests and latency distribution.

---

## ⚙️ Configuration Highlight (`prometheus.yml`)

The configuration leverages a `15s` scraping frequency for responsive updates:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'monitorPythonserver'
    static_configs:
      - targets: ['pythonserver:8080']
        labels:
          group: 'monitoring_python'

  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter1:9100']
        labels:
          group: 'monitoring_node_ex1'
      # Additional node targets...
```

---

## 📜 License
Distributed under the MIT License. See `LICENSE` for more information.
