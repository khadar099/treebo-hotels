
**ClusterIP → internal communication

NodePort → expose using node port

LoadBalancer → public access in cloud

ExternalName → external DNS mapping

Headless → direct pod access**


**1. ClusterIP (Default)**

Purpose: Expose a service internally inside the cluster.

Key points

Default service type.

Only accessible within the Kubernetes cluster.

Gets an internal cluster IP.

Used for microservice-to-microservice communication.

Example Use Case

Frontend service calling a backend API.

Backend calling a database.

Example YAML

apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 8080

Architecture

Pod → Service (ClusterIP) → Pod

**2. NodePort**

Purpose: Expose a service externally using a node's IP and port.

Key points

Opens a port (30000–32767) on every node.

External users can access it via:

<NodeIP>:<NodePort>

Example Use Case

Quick testing

Development environments

Accessing apps without a load balancer

Example YAML

spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30007

Access

http://NodeIP:30007

Architecture

Internet → NodeIP:NodePort → Service → Pod

**3. LoadBalancer**

Purpose: Expose service externally using a cloud provider load balancer.

Key points

Works in cloud environments (AWS, Azure, GCP).

Automatically provisions a cloud load balancer.

Gets a public IP.

Example Use Case

Production web applications

Public APIs

Example YAML

spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 8080

Architecture

Internet → Cloud LoadBalancer → NodePort → Pods

**4. ExternalName**

Purpose: Map a Kubernetes service to an external DNS name.

Key points

No proxying.

Returns a CNAME DNS record.

Example Use Case

Connecting to external databases

External APIs

Legacy systems outside the cluster

Example YAML

spec:
  type: ExternalName
  externalName: api.example.com

Result

myservice.default.svc.cluster.local → api.example.com

**5. Headless Service**

Purpose: Expose Pods directly without load balancing.

Key points

clusterIP: None

DNS returns individual Pod IPs.

Used with StatefulSets.

Example Use Case

Databases (MySQL cluster, Cassandra)

Kafka

Stateful applications

Example YAML

spec:
  clusterIP: None

Architecture

Client → DNS → Pod1 / Pod2 / Pod3
