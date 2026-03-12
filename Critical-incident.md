**Incident**  -  One of our hotel booking microservices went down in production, causing some users to fail bookings.


**1. We noticed alerts from Grafana and Prometheus indicating Booking Service errors and high latency, while customer complaints started coming in.”
2. I first checked EKS pod status and logs in Kibana to identify the source of the failures. It looked like the Booking Service pods were crashing due to memory exhaustion and database connection timeouts.
3. To quickly restore service, I scaled the Booking Service deployment and restarted pods while ensuring the EKS cluster had enough nodes via Cluster Autoscaler
4. After stabilizing the service, I reviewed Prometheus metrics and Kibana logs. We identified that a recent deployment caused higher memory usage, and DB queries were taking longer than expected, causing pods to crash
5. After stabilizing the service, I reviewed Prometheus metrics and Kibana logs. We identified that a recent deployment caused higher memory usage, and DB queries were taking longer than expected, causing pods to crash
6. We optimized queries, adjusted pod resource limits, tuned HPA and Cluster Autoscaler, and set up stricter alert thresholds to prevent recurrence.”
7. We conducted a postmortem, documented the incident, shared learnings with the team, and added resource and stress tests in CI/CD to avoid similar outages.**




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

**if interview ask how did you do then explain below**

How I Restored the Service in a Critical Incident (Step by Step)
1️⃣ Check the affected pods

First, I listed all pods in the prod namespace to see their status:

kubectl get pods -n prod

Observed which pods were CrashLoopBackOff or OOMKilled.

2️⃣ Describe pods for details

To understand why the pods were failing, I ran:

kubectl describe pod booking-service-xyz -n prod

This showed container termination reasons (e.g., memory limits, DB connection errors) and events.

3️⃣ Restart affected pods

To immediately bring back availability:

kubectl rollout restart deployment booking-service -n prod

This created new pods with fresh containers without downtime for other services.

4️⃣ Scale deployment to handle load

While restarting, I increased the replicas temporarily to handle user requests:

kubectl scale deployment booking-service --replicas=5 -n prod

This ensured enough pods were running while others restarted.

5️⃣ Check EKS node capacity

Verified if the cluster had enough nodes to schedule new pods:

kubectl get nodes
kubectl top nodes

Noticed that some nodes were almost full, so I ensured Cluster Autoscaler was enabled and could scale up nodes automatically.

In urgent cases, you can also manually increase node group size in AWS console or via eksctl:

eksctl scale nodegroup --cluster treebo-prod --name worker-ng-1 --nodes 5
6️⃣ Verify service is back

Checked the deployment status and logs to confirm recovery:

kubectl get pods -n prod
kubectl logs -f booking-service-xyz -n prod

Metrics in Prometheus/Grafana returned to normal.

7️⃣ Post-recovery

Once service was stable, I analyzed root cause and implemented permanent fixes (resource limits, query optimization, HPA tuning).



**Step 3 – Immediate Mitigation**

Prevent further customer impact:

Scale up replicas manually:

kubectl scale deployment booking-service --replicas=5 -n prod

Restart affected pods to restore availability:

kubectl rollout restart deployment booking-service -n prod

Enable temporary node autoscaling if cluster nodes are insufficient (Cluster Autoscaler triggered manually).

**Interview phrasing:**

**“To quickly restore service, I scaled the Booking Service deployment and restarted pods while ensuring the EKS cluster had enough nodes via Cluster Autoscaler.”**

if interveiwe asks how did you do this 

Step-by-Step: How I Restored the Booking Service

Checked pod status in EKS

Listed all pods in the production namespace to identify failures:

kubectl get pods -n prod

Found booking-service pods in CrashLoopBackOff due to memory limits and DB timeouts.

Examined pod details

Ran:

kubectl describe pod booking-service-xyz -n prod

Checked container events, logs, and reason for termination.

Restarted the deployment

Rolled out a restart to recreate pods without downtime:

kubectl rollout restart deployment booking-service -n prod

Scaled the deployment to handle load

Temporarily increased replicas to ensure enough pods were running:

kubectl scale deployment booking-service --replicas=5 -n prod

Checked cluster node capacity

Verified if there were enough EKS nodes to schedule new pods:

kubectl get nodes
kubectl top nodes

Cluster Autoscaler ensured new nodes were added automatically if pods couldn’t be scheduled.

Verified recovery

Monitored pods and logs to confirm they were running normally:

kubectl get pods -n prod
kubectl logs -f booking-service-xyz -n prod

Checked metrics in Prometheus/Grafana — CPU, memory, and error rates normalized.




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
