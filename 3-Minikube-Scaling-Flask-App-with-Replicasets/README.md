# Scaling a Flask App on Minikube with ReplicaSets

This exercise demonstrates Kubernetes ReplicaSets by running a Flask flash-sale
application on a single-node Minikube cluster, scaling it up, and observing
self-healing and pod distribution.

## Objective

* Understand ReplicaSets and Pods.
* Scale a Flask app deployment across multiple pods.
* Observe pod self-healing when a pod is deleted.
* Observe pod distribution across nodes.

## Technologies Used

* Docker
* Python
* Flask
* Kubernetes (Minikube)

## Files

```text
.
├── app.py
├── Dockerfile
├── flashsale-replicaset.yaml
└── README.md
```

## Steps Followed

### 1. Checked Docker and Minikube

Verified Docker Desktop was running, and reset any existing cluster:

```powershell
minikube stop
minikube delete
```

Hit `Access is denied` on `id_rsa.pub` during delete, caused by a locked file from a
running process. Closed other terminals holding the `.minikube` folder open and ran
`minikube delete` again, which completed successfully.

### 2. Started a Single-Node Minikube Cluster

```powershell
minikube start --nodes=1 --driver=docker
kubectl get nodes
```

```text
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   42s   v1.31.0
```

### 3. Created the Flask Application

The Flask app exposes `/`, `/buy`, and `/health` endpoints and returns the serving
pod's hostname in each response so load distribution can be observed.

`app.py`:

```python
from flask import Flask, request
import socket, time, random

app = Flask(__name__)

@app.get("/")
def homepage():
    return {"message": "Welcome to Big Sale!", "pod": socket.gethostname(), "ts": time.time()}

@app.get("/buy")
def buy():
    item = random.choice(["Smartphone", "Shoes", "Headphones", "Laptop"])
    user = request.args.get("user", f"user{random.randint(1,1000)}")
    return {"status": "success", "item": item, "user": user,
            "served_by_pod": socket.gethostname(), "time": time.strftime("%H:%M:%S")}

@app.get("/health")
def health():
    return {"status": "healthy", "pod": socket.gethostname()}
```

### 4. Created the Dockerfile

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY app.py .
RUN pip install --no-cache-dir flask gunicorn
CMD ["gunicorn","-b","0.0.0.0:5000","app:app","--workers","1","--threads","2"]
```

### 5. Built the Docker Image Inside Minikube

Building inside Minikube's own Docker daemon avoids relying on Docker Hub.

```powershell
& minikube -p minikube docker-env --shell powershell | Invoke-Expression
docker build -t flashsale:1.0 .
```

Both commands were run in the same PowerShell window, since the Docker environment
variables only apply to the window they're set in.

### 6. Created the ReplicaSet and Service Manifest

`flashsale-replicaset.yaml` defines a ReplicaSet (`flashsale-rs`, 3 replicas) with
readiness and liveness probes on `/health`, and a ClusterIP Service (`flashsale-svc`)
exposing port 80 → 5000.

### 7. Applied the Manifest

```powershell
kubectl apply -f flashsale-replicaset.yaml
```

```text
replicaset.apps/flashsale-rs created
service/flashsale-svc created
```

### 8. Verified the Pods

```powershell
kubectl get pods
kubectl get rs
```

Some pods initially showed `ImagePullBackOff`, since they were scheduled before the
image had finished loading into Minikube's Docker daemon. They recovered
automatically once the image was available; `kubectl delete pod <name>` was used to
force an immediate retry on ones that hadn't yet.

```text
NAME                 READY   STATUS    RESTARTS   AGE
flashsale-rs-464dv   1/1     Running   0          3m35s
flashsale-rs-s9kfq   1/1     Running   0          3m35s
flashsale-rs-tkblx   1/1     Running   0          3m35s

NAME           DESIRED   CURRENT   READY   AGE
flashsale-rs   3         3         3       40s
```

### 9. Scaled the ReplicaSet to 5 Replicas

```powershell
kubectl scale rs flashsale-rs --replicas=5
kubectl get rs
kubectl get pods
```

```text
NAME           DESIRED   CURRENT   READY   AGE
flashsale-rs   5         5         5       7m38s
```

2 new pods were created to bring the total to 5.

### 10. Tested Self-Healing

Deleted one pod and confirmed a replacement appeared automatically:

```powershell
kubectl delete pod flashsale-rs-464dv
kubectl get pods
```

A new pod with a different name appeared, and the total remained at 5.

### 11. Verified Pod Distribution Across Nodes

```powershell
kubectl get pods -o wide
kubectl get nodes
```

All 5 pods ran on the single `minikube` node, since the cluster has only one node.

### 12. Accessed the Flask App Locally

The Service is `ClusterIP`, so it isn't reachable directly from the host. Used
port-forwarding instead:

```powershell
kubectl port-forward svc/flashsale-svc 8080:80
```

Opened the following URLs in the browser:

```text
http://localhost:8080/
http://localhost:8080/buy
http://localhost:8080/health
```

### 13. Observed Load Distribution Across Pods

Port-forwarding pins to a single pod, so `served_by_pod` didn't change across
requests. To see requests spread across pods, traffic was sent from inside the
cluster, through the Service:

```powershell
kubectl run curl-test --rm -it --image=curlimages/curl --restart=Never -- `
  sh -c "for i in 1 2 3 4 5 6 7 8 9 10; do curl -s flashsale-svc/buy; echo; done"
```

Different pod names appeared in `served_by_pod` across the ten responses, confirming
requests were being load-balanced across the ReplicaSet's pods.

### 14. Cleaned Up

```powershell
kubectl delete -f flashsale-replicaset.yaml
minikube stop
```

## Result

The exercise successfully demonstrated:

* Creation and scaling of a Kubernetes ReplicaSet.
* Automatic pod replacement (self-healing) when a pod is deleted.
* All pods scheduled onto the single available node.
* Access to a ClusterIP Service via `kubectl port-forward`.
* Load distribution across pods observed via in-cluster requests to the Service.