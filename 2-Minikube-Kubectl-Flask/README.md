# Flask App on Minikube (Kubernetes)

A simple Flask application deployed to a local Kubernetes cluster using Minikube, kubectl, and Docker.

## Overview

This project demonstrates the complete workflow of containerizing a Python Flask application using Docker, deploying it to a local Kubernetes cluster using Minikube, and exposing it using a Kubernetes NodePort Service.

## Technologies Used

- Python
- Flask
- Docker
- Kubernetes
- Minikube
- kubectl

## Project Structure

    2-Minikube-Kubectl-Flask/
    ├── app.py
    ├── Dockerfile
    ├── flask-deployment.yaml
    └── README.md

## Prerequisites

Make sure the following are installed:

- Minikube
- kubectl
- Docker
- Python 3.8+ (optional for local testing)

Check the installations:

    minikube version
    kubectl version --client
    docker --version
    python --version

## Application

The Flask application contains a single route (`/`) that returns:

    Hello from Flask on Kubernetes!

The application runs on port `15000` and is bound to `0.0.0.0` so that it can receive connections from outside the container.

## Docker Configuration

The Dockerfile:

- Uses `python:3.8-slim` as the base image.
- Installs Flask.
- Copies the Flask application into the container.
- Starts the Flask application when the container starts.

The Docker image is named:

    flask-app:latest

## Kubernetes Configuration

The `flask-deployment.yaml` file contains two Kubernetes resources:

### Deployment

The Deployment:

- Runs 1 replica of the Flask application.
- Uses the `flask-app:latest` Docker image.
- Uses `imagePullPolicy: Never`.
- Runs the application inside a Kubernetes Pod.

The `imagePullPolicy: Never` setting ensures Kubernetes uses the Docker image available inside Minikube instead of trying to pull it from a container registry.

### Service

The Service uses the `NodePort` type to expose the Flask application.

    Service Port: 15000
    Target Port: 15000

Traffic flow:

    Host Machine
         |
         v
    Minikube NodePort
         |
         v
    Kubernetes Service
         |
         v
    Flask Pod
         |
         v
      Port 15000

## Setup and Deployment

### 1. Start Minikube

Start the local Kubernetes cluster:

    minikube start

Check the cluster status:

    minikube status

### 2. Configure Docker to Use Minikube

For PowerShell, run:

    & minikube -p minikube docker-env --shell powershell | Invoke-Expression

This configures the current terminal session to use Minikube's Docker daemon.

Note: This configuration applies only to the current terminal session. If you open a new terminal, run the command again before building the Docker image.

Verify Docker:

    docker info

### 3. Build the Docker Image

From the project directory, run:

    docker build -t flask-app .

Verify the image:

    docker images

You should see the `flask-app` image.

### 4. Deploy to Kubernetes

Apply the Kubernetes configuration:

    kubectl apply -f flask-deployment.yaml

### 5. Verify the Deployment

Check the Deployment:

    kubectl get deployments

Check the Pods:

    kubectl get pods -l app=flask-app

Check the Service:

    kubectl get services

The Pod should eventually show:

    Running

### 6. Access the Application

Run:

    minikube service flask-app-service --url

Minikube will return a URL similar to:

    http://127.0.0.1:xxxxx

Keep this terminal open.

Open a new terminal and run:

    curl.exe http://127.0.0.1:<port-shown>

Expected output:

    Hello from Flask on Kubernetes!

You can also open the URL provided by Minikube in a web browser.

## Useful Commands

    minikube status
    minikube start
    minikube stop
    minikube delete

    kubectl get nodes
    kubectl get pods
    kubectl get deployments
    kubectl get services

    kubectl logs <pod-name>
    kubectl describe pod <pod-name>
    kubectl describe deployment flask-app

    kubectl delete -f flask-deployment.yaml

## Troubleshooting

### ImagePullBackOff or ErrImagePull

If the Pod shows `ImagePullBackOff` or `ErrImagePull`, make sure the Docker image was built inside Minikube's Docker environment.

Run:

    & minikube -p minikube docker-env --shell powershell | Invoke-Expression

Then rebuild the image:

    docker build -t flask-app .

Also make sure the Deployment contains:

    imagePullPolicy: Never

### curl to Port 15000 Fails

Running:

    curl.exe http://127.0.0.1:15000

may fail.

This is expected because port `15000` is the container/Service port and is not directly exposed to the host.

Instead, use:

    minikube service flask-app-service --url

Then access the URL provided by Minikube.

### Pod Is Not Running

Check the Pods:

    kubectl get pods

Get detailed information:

    kubectl describe pod <pod-name>

Check application logs:

    kubectl logs <pod-name>

### After Restarting Minikube

Start Minikube:

    minikube start

Check the resources:

    kubectl get pods
    kubectl get deployments
    kubectl get services

If necessary, reapply the configuration:

    kubectl apply -f flask-deployment.yaml

If the Pod shows `ImagePullBackOff`, configure Docker again and rebuild the image:

    & minikube -p minikube docker-env --shell powershell | Invoke-Expression

    docker build -t flask-app .

## Cleanup

Delete the Kubernetes resources:

    kubectl delete -f flask-deployment.yaml

Stop Minikube:

    minikube stop

Delete the Minikube cluster completely:

    minikube delete

## Complete Workflow

    Flask Application
           |
           v
       Dockerfile
           |
           v
     Docker Image
     flask-app:latest
           |
           v
        Minikube
           |
           v
    Kubernetes Deployment
           |
           v
          Pod
           |
           v
    Kubernetes Service
        (NodePort)
           |
           v
      Host Machine
           |
           v
       Web Browser

## Expected Result

When everything is configured correctly, opening the Minikube service URL should display:

    Hello from Flask on Kubernetes!

## Learning Outcomes

This project provides hands-on experience with:

- Flask application development
- Docker containerization
- Docker image creation
- Kubernetes Deployments
- Kubernetes Services
- Minikube
- kubectl commands
- NodePort networking
- Container debugging
- Local Kubernetes deployment