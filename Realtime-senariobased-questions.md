**Scenario / Question	Concise Answer (Stepwise / Bullet Points)**

1.** Booking Service Pod Crashes	- kubectl get pods → identify CrashLoopBackOff**

- kubectl describe pod → check logs/events
- 
- Restart deployment: kubectl rollout restart
- 
- Scale replicas temporarily: kubectl scale deployment
- 
- Check node capacity / Cluster Autoscaler
- 
- Postmortem: optimize queries, adjust resource limits, tune HPA
  


**2. High Latency Payment Service	- Check Grafana & Prometheus metrics**

- Use OpenTelemetry traces to find slow calls
- 
- Check Kibana logs for retries/timeouts
- 
- Temporary mitigation: scale pods, increase timeouts
- 
- Permanent fix: caching, circuit breaker, optimized code

3. **Database Connection Pool Exhaustion	- Detect via Prometheus DB metrics**
  
- Temporarily scale DB or restart pods

- Root cause: low connection pool, high traffic

- Permanent: increase pool, optimize queries, add retry/backoff, alerts

**4. Failed ArgoCD Deployment	- Check ArgoCD status & kubectl describe deployment**

- Rollback: kubectl rollout undo
  
- Debug pipeline/Helm chart issues
  
- Permanent fix: pre-deployment tests, staging validation, sync hooks
  
**5. Sudden High Traffic (Black Friday)	- Detect via Prometheus/Grafana CPU/memory spikes**

- Temporarily scale pods (kubectl scale)
  
- Ensure Cluster Autoscaler adds nodes
  
- Root cause: HPA thresholds too high, node limits
  
- Permanent: tune HPA, predictive scaling, increase max nodes
  
**6. Prometheus Stops Scraping Metrics	- Prometheus UI → check targets**
   
- Verify /metrics endpoint
  
- Check service discovery/DNS in Kubernetes
  
- Restart Prometheus/OTel Collector
  
- Confirm metrics reappear in dashboards
  
**7. HashiCorp Vault Secret Misconfigured	- Rotate affected secrets safely**
  
- Restart pods consuming secrets
  
- Verify access works
  
- Permanent: pipeline for automatic secret rotation, monitoring for expired/missing secrets
  
**8. Docker Image Vulnerability Detected (Trivy)	- Block deployment of vulnerable image**
  
- Patch or rebuild image
  
- Redeploy new version
  
- Permanent: Integrate Trivy/other scans in CI/CD, prevent critical vulnerabilities in production
  
**9. Slow Booking Confirmation	- Use OpenTelemetry traces to pinpoint slow microservice or DB query**
  
- Check logs in Kibana for errors
  
- Optimize code or query
  
- Monitor metrics for improvement
  
**10. EKS Node Failure / Pod Eviction	- Check pod status (kubectl get pods)**
  
- Pods rescheduled automatically via deployment/HPA
  
- Verify Cluster Autoscaler added nodes
  
- Permanent: ensure multi-AZ deployment, HPA tuning, pod disruption budgets
  
**11. CI/CD Pipeline Failure (Jenkins/ArgoCD)	- Check Jenkins build logs and ArgoCD sync logs**

- Rollback deployment if needed
  
- Fix pipeline scripts, image tags, Helm charts
  
- Permanent: staging validation, automated tests, notifications on failure
  
**12. Metrics/Logs Missing or Delayed	- Check Prometheus scraping and OTel Collector logs**

- Check Filebeat → Elasticsearch pipeline
  
- Fix network/DNS or restart components
  
- Add monitoring/alerts for missing data
