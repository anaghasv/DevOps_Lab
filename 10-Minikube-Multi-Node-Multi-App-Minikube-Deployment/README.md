# Minikube Multi-Node Cluster with Multiple Applications and ReplicaSets

## Overview

This project demonstrates deploying multiple Flask applications on a multi-node Kubernetes cluster using Minikube. The Product Catalog application runs with 2 replicas, while the Shopping Cart application runs with 3 replicas. Kubernetes Deployments, ReplicaSets, Services, and pod anti-affinity rules are used to manage and distribute application pods.

## Technologies Used

- **Minikube** – Multi-node Kubernetes cluster
- **Kubernetes** – Container orchestration and replica management
- **Docker** – Container image creation
- **Python** – Application development
- **Flask** – Web application framework
- **YAML** – Kubernetes configuration

## Project Structure

```text
10-Minikube-Multi-Node-Multi-App-Minikube-Deployment/
├── product_catalog.py
├── shopping_cart.py
├── Dockerfile.product
├── Dockerfile.shopping
├── product_catalog_deployment.yaml
├── shopping_cart_deployment.yaml
└── README.md
```

## Applications

- **Product Catalog (AppA):** Flask application with a `/products` endpoint and 2 replicas.
- **Shopping Cart (AppB):** Flask application with a `/cart` endpoint and 3 replicas.

## Implementation Steps

### 1. Start the Minikube Cluster

Create a three-node Minikube cluster named `devops-multinode`.

```bash
minikube start --nodes 3 -p devops-multinode --force
```

Verify the cluster nodes:

```bash
kubectl get nodes
```

### 2. Build Docker Images

Build the Product Catalog image:

```bash
docker build -t product-catalog:latest -f Dockerfile.product .
```

Build the Shopping Cart image:

```bash
docker build -t shopping-cart:latest -f Dockerfile.shopping .
```

### 3. Load Images into Minikube

Load the Docker images into the Minikube cluster:

```bash
minikube -p devops-multinode image load product-catalog:latest
minikube -p devops-multinode image load shopping-cart:latest
```

Verify the images inside the Minikube node:

```bash
minikube -p devops-multinode ssh -- docker images
```

### 4. Deploy the Product Catalog Application

Apply the Kubernetes Deployment configuration:

```bash
kubectl apply -f product_catalog_deployment.yaml
```

### 5. Deploy the Shopping Cart Application

Apply the Kubernetes Deployment configuration:

```bash
kubectl apply -f shopping_cart_deployment.yaml
```

Both deployment YAML files should use a namespace that exists in the cluster. This exercise uses the `default` namespace unless another namespace has been created.

### 6. Verify Deployments and Replicas

Check the deployments:

```bash
kubectl get deployments
```

Check the ReplicaSets:

```bash
kubectl get replicasets
```

Check the pods and their node assignments:

```bash
kubectl get pods -o wide
```

Check the services:

```bash
kubectl get services
```

Expected replica configuration:

| Application | Replicas |
|---|---:|
| Product Catalog | 2 |
| Shopping Cart | 3 |
| Total | 5 |

### 7. Access the Applications

Access the Product Catalog service:

```bash
minikube -p devops-multinode service product-catalog-service
```

Access the Shopping Cart service:

```bash
minikube -p devops-multinode service shopping-cart-service
```

Use the actual service names defined in the Kubernetes YAML files if they differ from these examples.

### 8. Test the Applications

The Product Catalog application exposes the `/products` endpoint. The Shopping Cart application exposes the `/cart` endpoint.

If the services are configured with NodePort, test the endpoints using the URLs provided by Minikube:

```text
http://<product-catalog-service-url>/products
http://<shopping-cart-service-url>/cart
```

Replace the placeholders with the actual service URLs. The Shopping Cart application also supports POST requests if implemented in its Flask code.

### 9. Verify Pod Distribution

Run:

```bash
kubectl get pods -o wide
```

Inspect the `NODE` column to see where each pod is running. Pod anti-affinity rules influence whether pods are scheduled on different nodes. Actual placement depends on the rules configured in the YAML files and available cluster resources.

## Kubernetes Configuration

The deployment YAML files define:

- Application container images.
- Desired replica counts.
- Pod labels and selectors.
- Pod anti-affinity rules.
- Kubernetes Services for application access.

The Product Catalog deployment uses 2 replicas, and the Shopping Cart deployment uses 3 replicas.

## Cleanup

Delete the Kubernetes deployments and services:

```bash
kubectl delete -f product_catalog_deployment.yaml
kubectl delete -f shopping_cart_deployment.yaml
```

Stop the Minikube cluster:

```bash
minikube -p devops-multinode stop
```

Delete the cluster when it is no longer required:

```bash
minikube -p devops-multinode delete
```

## Learning Outcomes

- Creating a multi-node Minikube cluster.
- Building and loading Docker images.
- Deploying Flask applications using Kubernetes.
- Managing Deployments, ReplicaSets, and replicas.
- Configuring pod anti-affinity for pod scheduling.
- Exposing applications through Kubernetes NodePort Services.
- Verifying pod status and node distribution using `kubectl`.

## Result

Successfully configured a multi-node Minikube cluster and deployed the Product Catalog and Shopping Cart applications with multiple replicas and Kubernetes Services.