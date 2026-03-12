<img width="1511" height="850" alt="image" src="https://github.com/user-attachments/assets/852d99ef-b836-4adf-bd7e-318296e66edc" />


**1. Control Plane Components**

The Control Plane manages the cluster’s overall state, scheduling, and operations.

**a) kube-apiserver**

Acts as the front-end for the Kubernetes control plane.

Exposes the Kubernetes API, which is used by all components to communicate.

All interactions (kubectl commands, REST calls) go through the API server.

Ensures security via authentication, authorization, and admission control.

**b) etcd**

Distributed key-value store used to store all cluster data, including configurations, states, and secrets.

Highly available and consistent.

Serves as the source of truth for the cluster.

**c) kube-scheduler**

Watches for newly created pods that do not have a node assigned.

Schedules pods onto appropriate nodes based on resource requirements, constraints, and policies.

**d) kube-controller-manager**

Runs various controller processes that regulate the cluster state.

**Examples:**

Node Controller – monitors node health.

Replication Controller – ensures desired number of pod replicas.

Endpoint Controller – manages services and endpoints.

Ensures desired state matches the actual state.

**e) cloud-controller-manager (optional)**

Integrates Kubernetes with the underlying cloud provider (AWS, Azure, GCP).

Handles cloud-specific controllers like LoadBalancer, Node management, and storage provisioning.

**2. Node (Worker) Components**

Nodes are the machines (physical or virtual) that run your application workloads (pods).

**a) kubelet**

An agent that runs on every node.

Ensures containers described in pods are running.

Communicates with the API server to get pod specs and report node status.

**b) kube-proxy**

Maintains network rules for pods and services.

Implements service discovery and load balancing across pods.

Handles traffic routing inside the cluster.

**c) Container Runtime**

Software responsible for running containers on a node.

Examples: Docker, containerd, CRI-O.

kubelet interacts with the container runtime to manage container lifecycle.

**3. Additional Concepts**

Pods – Smallest deployable unit in Kubernetes (one or more containers with shared networking and storage).

Services – Abstracts a set of pods as a network service.

Namespaces – Logical partitions to isolate resources in a cluster.

ConfigMaps & Secrets – For injecting configuration and sensitive data into pods.

Architecture Overview Diagram

**Here’s a simplified view:**

+-------------------+       +-----------------+
|   Control Plane   |       |      Nodes      |
|-------------------|       |----------------|
| kube-apiserver    | <----> | kubelet       |
| etcd              |       | kube-proxy     |
| kube-scheduler    |       | Container Runtime |
| kube-controller   |       | Pods/Containers |
| cloud-controller  |       |                 |
+-------------------+       +-----------------+

Control Plane: Maintains cluster state, schedules workloads, handles management.

Nodes: Run workloads (pods), report status, handle networking.
