# Kubernetes WordPress Deployment & Monitoring Workshop

## Overview
This project demonstrates a full migration of a WordPress application from Docker Compose to a production-ready Kubernetes environment. It includes a resilient architecture with persistent storage, automated deployment using Helm, and a complete observability stack.

## Architecture
The system is built with high availability and data persistence in mind:
- **WordPress:** Deployed as a `Deployment` with 2 replicas to ensure service availability.
- **Database:** MariaDB deployed as a `StatefulSet` with a `PersistentVolumeClaim (PVC)` to guarantee data persistence and stable network identity.
- **Ingress:** NGINX Ingress Controller manages external traffic, providing a single point of entry.
- **Monitoring:** Integrated `kube-prometheus-stack` (Prometheus & Grafana) for real-time cluster and application metrics.

## Prerequisites
- AWS EC2 Instance (t3.medium recommended)
- Minikube installed and running
- Helm 3.x
- Configured Amazon ECR repository

## Project Structure
```text
.
├── wordpress/               # Custom Helm chart for the application
│   ├── templates/           # Deployment, StatefulSet, Services, and Ingress
│   └── values.yaml          # Application configuration
├── nginx-ingress/           # NGINX Ingress Controller Helm resources
└── README.md                # Project documentation

## Installation & Deployment

### 1. Setup Infrastructure
Ensure your Ingress Controller is running:
```bash
helm install ingress-nginx ./nginx-ingress -n ingress-nginx --create-namespace

### 2. Configure ECR Secret
Since the images are stored in a private Amazon ECR, create a pull secret:

Bash
kubectl create secret docker-registry regcred \
  --docker-server=${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com \
  --docker-username=AWS \
  --docker-password=$(aws ecr get-login-password) \
  -n wordpress-app

### 3. Deploy WordPress & MariaDB
```bash
kubectl create ns wordpress-app
helm install my-wordpress ./wordpress -n wordpress-app

## Monitoring & Observability
We use Prometheus for scraping metrics and Grafana for visualization.

### Custom Uptime Panel
A custom Grafana panel was created to monitor container uptime.
- **Metric used:** `time() - kube_pod_start_time{namespace="wordpress-app"}`
- **Visualization:** Stat panel with `duration` units to show exactly how long each container has been running.
