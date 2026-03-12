**Pull Model Flow for Metrics**

In the pull model:

Microservices expose metrics on a dedicated endpoint, usually /metrics.

Examples: CPU usage, memory, request latency, custom app metrics (bookings/minute).

Each microservice runs in a pod in EKS.

Prometheus scrapes these endpoints at a fixed interval (e.g., every 15 seconds).

Prometheus queries each microservice directly or through the OTel Collector if it’s being used to standardize metrics.

Each scrape pulls the current snapshot of metrics.

Metrics are stored as time-series data with labels:

service=booking-service

namespace=prod

pod=booking-service-xyz

Grafana queries Prometheus using PromQL to visualize metrics:

Example dashboards:

CPU and memory per microservice

Request latency and error rates



**Pull Model Diagram (Text Version)**

MICROSERVICES (Booking, Payment, Inventory, etc.)


       | expose /metrics
       
       v

   
+-------------------+
| Prometheus Server |  <-- Scrapes metrics at regular intervals
+-------------------+


      |
      v

    
+-------------------+
| Grafana           |  <-- Queries Prometheus via PromQL and visualizes dashboards
+-------------------+



**How to Explain It in Interviews**

“In our pull model setup, each microservice exposes a /metrics endpoint with performance and application metrics. 
Prometheus scrapes these endpoints at defined intervals, storing metrics as time-series data with labels like service, namespace, and pod.
Grafana connects to Prometheus as a data source and visualizes these metrics in dashboards. 
This allows us to monitor resource usage, request latency, error rates, and application-specific KPIs in real time.
Using the pull model ensures Prometheus has full control over scraping intervals and reduces the risk of overloading services with push traffic.”

Booking success/failure rates

Alerts can be configured if thresholds are breached.
