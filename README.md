# Project Deployment Guide

This document outlines the steps to build and deploy a project using Jenkins on Kubernetes. The project is exposed through Nginx Ingress, with SSL certificates provided by CertManager. System monitoring is handled by Prometheus and Grafana, while ELK (Elasticsearch, Logstash, Kibana) is used for logging.

## 1. Prerequisites

Before starting the build and deployment process, ensure you have the following tools and services in place:

- Kubernetes Cluster 
- Jenkins installed on the Kubernetes Cluster
- ArgoCD installed for GitOps-based deployment
- Nginx Ingress Controller for service exposure
- CertManager for automatic SSL certificate management
- Prometheus & Grafana for monitoring
- ELK Stack (Elasticsearch, Logstash, Kibana) for logging

## 2. Setup Jenkins on Kubernetes

### Step 1: Deploy Jenkins on Kubernetes

Use a Helm chart or a Kubernetes deployment YAML to deploy Jenkins on your cluster. For example, you can use the following Helm command:

```bash
helm install jenkins stable/jenkins --namespace devops
```

### Step 2: Expose Jenkins via Ingress (Optional)


### Step 3: Install Jenkins Plugins

Install necessary plugins for Kubernetes and Git integration:

- Kubernetes CLI Plugin
- Git Plugin
- Prometheus Plugin
- ArgoCD Plugin

### Step 4: Create Jenkins Pipeline (Jenkinsfile)

In your repository, create a `Jenkinsfile` inside the `devops` folder, which defines the pipeline steps for building and deploying the project. Example `Jenkinsfile`:


### Step 5: Set Up Jenkins Triggers

In Jenkins, set up a webhook trigger that listens for changes in your Git repository (e.g., GitHub or GitLab). This can be done by configuring a webhook in your Git repository settings to call the Jenkins job.

## 3. Deploy ArgoCD on Kubernetes

### Step 1: Install ArgoCD

Use the following command to deploy ArgoCD on Kubernetes:

```bash
kubectl create namespace devops
kubectl apply -n devops -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Step 2: Configure ArgoCD

Set up ArgoCD to automatically deploy your application. Application manifest file in folder devops/argocd.yaml

Apply this file to ArgoCD using:

```bash 
kubectl apply -f argocd.yaml -n devops
```

## 4. Nginx Ingress and SSL Certificate Setup

### Step 1: Install Nginx Ingress Controller

Deploy the Nginx Ingress Controller using the Helm chart:

```bash
helm install nginx-ingress stable/nginx-ingress --namespace ingress-nginx
```

### Step 2: Configure CertManager

Install CertManager:

```bash
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.3.1/cert-manager.crds.yaml
helm install cert-manager jetstack/cert-manager --namespace cert-manager
```

Create an Issuer resource to automatically generate SSL certificates:

```yaml
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: letsencrypt-prod
  namespace: default
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your-email@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
```
---

## 7. Automating Build and Deployment Process

### Step 1: Trigger Jenkins Build

Configure Jenkins to automatically trigger a build when code is pushed to the repository. This can be done via webhook configuration.

### Step 2: Trigger ArgoCD Deployment

Once Jenkins finishes the build, it will trigger ArgoCD to deploy the changes automatically to the Kubernetes cluster.

---

With this setup, you will have a complete CI/CD pipeline that builds, deploys, and monitors your application on Kubernetes. The use of Nginx Ingress ensures that your services are exposed securely, and CertManager automates SSL certificate management. Prometheus and Grafana provide detailed monitoring, while the ELK stack helps with log aggregation and visualization.