<img width="506" height="225" alt="image" src="https://github.com/user-attachments/assets/58ec067c-683d-4e1f-bcca-1d72096d84e9" />


3️⃣ Metric Flow (Step-by-Step)

Microservices Expose Metrics

Each microservice exposes /metrics with:

CPU, memory usage

Request count, latency, errors

Custom business metrics (e.g., bookings/minute)

Prometheus Scrapes Metrics (Pull Model)

Prometheus runs in EKS and is configured with scrape targets (microservices endpoints or OTel Collector).

Pull interval is configurable (default 15s).

Stores scraped metrics as time-series data with labels (service, pod, namespace).

Optional OpenTelemetry Collector

Can be used to:

Collect metrics from microservices or OTLP sources

Transform, batch, and standardize metrics

Expose metrics to Prometheus via /metrics endpoint

Grafana Visualization

Grafana queries Prometheus using PromQL

Displays dashboards with:

Pod-level CPU/memory usage

Request latency & error rates

Business metrics like booking counts

Supports alerts when thresholds are breached

4️⃣ Interview-Friendly Explanation

“In our EKS cluster, each microservice exposes metrics via a /metrics endpoint. Prometheus is deployed in the cluster and pulls metrics from these endpoints at regular intervals, storing them as time-series data. Optionally, we use the OpenTelemetry Collector as a DaemonSet to collect and standardize metrics, then expose them to Prometheus. Grafana connects to Prometheus as a data source and visualizes metrics in real time through dashboards. This architecture allows us to monitor both system and business metrics, set alerts, and scale our services confidently. Using the pull model ensures that Prometheus controls scrape frequency and avoids overloading the microservices.”
