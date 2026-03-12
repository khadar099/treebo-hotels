**Step-by-Step Approach to Handling a Critical Incident**

scenario in  EKS + microservices + CI/CD + DevSecOps environment:

One of our hotel booking microservices went down in production, causing some users to fail bookings.

**Step 1 – Detection & Alerting**

**How it started:**

Alerts triggered from Grafana dashboards (high error rate, HTTP 500s from Booking Service)

Prometheus showed CPU spike and latency increase



**Interview phrasing:**

“We noticed alerts from Grafana and Prometheus indicating Booking Service errors and high latency, while customer complaints started coming in.”

**Step 2 – Initial Assessment**

Checked logs and metrics immediately:

Accessed Kibana to look at Booking Service logs (via Filebeat → Elasticsearch)

Checked Pod status in EKS:

kubectl get pods -n prod
kubectl describe pod booking-service-xyz -n prod

**Identify symptoms:**

Pods in CrashLoopBackOff

OOMKilled / memory spikes

Errors connecting to SQL DB

**Interview phrasing:**

**“I first checked EKS pod status and logs in Kibana to identify the source of the failures. It looked like the Booking Service pods were crashing due to memory exhaustion and database connection timeouts.”**

**Step 3 – Immediate Mitigation**

Prevent further customer impact:

Scale up replicas manually:

kubectl scale deployment booking-service --replicas=5 -n prod

Restart affected pods to restore availability:

kubectl rollout restart deployment booking-service -n prod

Enable temporary node autoscaling if cluster nodes are insufficient (Cluster Autoscaler triggered manually).

**Interview phrasing:**

**“To quickly restore service, I scaled the Booking Service deployment and restarted pods while ensuring the EKS cluster had enough nodes via Cluster Autoscaler.”**

**Step 4 – Root Cause Analysis (Post-Incident)**

Check metrics history in Prometheus: CPU/memory spikes, request patterns

Analyze logs in Kibana: errors, DB timeouts, external API failures

Check recent deployments: Jenkins pipeline → ArgoCD logs

Find root cause:

New deployment increased memory usage

Some SQL queries were slow → connection pool exhaustion

**Interview phrasing:**

**“After stabilizing the service, I reviewed Prometheus metrics and Kibana logs. We identified that a recent deployment caused higher memory usage, and DB queries were taking longer than expected, causing pods to crash.”**

**Step 5 – Permanent Fix**

Code/Query Optimization:

Optimized SQL queries to reduce execution time

Adjust Pod Resource Limits:

Updated requests and limits in deployment to prevent OOMKilled pods

Update HPA / Cluster Autoscaler Config:

Ensure HPA triggers scaling before pods reach resource limits

Add alert thresholds:

CPU/memory thresholds, error rate alerts

**Interview phrasing:**

**“We optimized queries, adjusted pod resource limits, tuned HPA and Cluster Autoscaler, and set up stricter alert thresholds to prevent recurrence.”
**
**Step 6 – Postmortem & Learnings**

Document incident: timeline, root cause, resolution

Share learnings with team: code optimization, resource planning

Improve CI/CD pipeline: add stress tests or memory checks

**Interview phrasing:**

**“We conducted a postmortem, documented the incident, shared learnings with the team, and added resource and stress tests in CI/CD to avoid similar outages.”**

**✅ Key Points to Emphasize in Interview**

Detection & triage – You noticed metrics, logs, and alerts.

Immediate mitigation – Restored service quickly to minimize user impact.

Root cause analysis – Used Prometheus + Grafana + Kibana + EKS to find cause.

Permanent fix – Code, resource limits, autoscaling, and monitoring improvements.

Learning & documentation – Showed responsibility and continuous improvement.
