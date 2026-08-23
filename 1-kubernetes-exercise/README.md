# Kubernetes Exercise 1 – Hello Pod

## 📌 Objective

The objective of this exercise is to understand the basics of Kubernetes by deploying an Nginx container as a Pod using Minikube and exposing it through a NodePort Service.

---

## 💼 Business Problem

Imagine working as a **DevOps Engineer at Zepto**. The product team has developed a lightweight web application that displays the storefront and delivery status page for customers.

The task is to deploy this application on Kubernetes so that it is:

* Highly available
* Portable
* Easy to manage
* Scalable for future requirements

For this exercise, the **Nginx container** is used to simulate the Zepto web application.

---

## 🛠️ Technologies Used

| Technology         | Purpose                      |
| ------------------ | ---------------------------- |
| **Kubernetes**     | Container orchestration      |
| **Minikube**       | Runs Kubernetes locally      |
| **Docker Desktop** | Container runtime            |
| **kubectl**        | Kubernetes command-line tool |
| **Nginx**          | Sample web application       |

---

## 🚀 Implementation

### Step 1: Start Minikube

Start a local Kubernetes cluster using Docker as the driver.

```bash
minikube start --driver=docker
```

### Step 2: Create the Nginx Pod

Create a Pod named `hello-k8s` using the Nginx image.

```bash
kubectl run hello-k8s --image=nginx --port=80
```

### Step 3: Verify the Pod

Check whether the Pod is running.

```bash
kubectl get pods
```

### Step 4: Expose the Pod

Create a NodePort Service to expose the Nginx application.

```bash
kubectl expose pod hello-k8s --type=NodePort --port=80
```

### Step 5: Verify the Service

```bash
kubectl get services
```

### Step 6: Check the Kubernetes Node

```bash
kubectl get nodes
```

### Step 7: Access the Application

Open the Nginx application using Minikube.

```bash
minikube service hello-k8s
```



## ✅ Result

The Nginx application was successfully deployed as a Kubernetes Pod using Minikube.

The Pod was exposed through a **NodePort Service**, and the Nginx Welcome Page was successfully accessed through the browser.

---

## 🎯 Learning Outcomes

Through this exercise, I learned how to:

* Create and manage a Kubernetes Pod
* Deploy a container using Kubernetes
* Use Minikube to run Kubernetes locally
* Expose an application using a NodePort Service
* Check Kubernetes Pod, Service, and Node status
* Access a Kubernetes application from a browser

---

## 📁 Project Structure

```text
1-kubernetes-exercise/
│
├── README.md
├── commands.txt
├── output-snippets.txt
│
└── screenshots/
    ├── image.png
    ├── image2.png
    ├── image3.png
    ├── image4.png
    └── image5.png
```

---

## 👨‍💻 Exercise

**Kubernetes Hands-On Exercise Series – Exercise 1**

**Topic:** Hello Pod
**Application:** Nginx
**Environment:** Local Kubernetes Cluster using Minikube
