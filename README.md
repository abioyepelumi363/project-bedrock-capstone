# Project Bedrock: Cloud-Native Retail Store Deployment

## Project Overview
This project demonstrates a production-grade deployment of a microservices e-commerce application on AWS. I used **Terraform** to provision the infrastructure (EKS, VPC, Networking) and **Kubernetes** to orchestrate the application services.

The application is the AWS Retail Store Sample App (https://github.com/aws-containers/retail-store-sample-app), which simulates a real-world shopping platform with services for catalog, cart, checkout, and orders.

## Architecture Highlights
* **Infrastructure as Code:** Fully automated infrastructure setup using Terraform.
* **Container Orchestration:** AWS EKS (Elastic Kubernetes Service) for managing microservices.
* **Networking:** Custom VPC with public/private subnets and an Internet Gateway.
* **Accessibility:** Application exposed via an AWS Load Balancer.

## Prerequisites
* AWS CLI (Configured)
* Terraform installed
* kubectl installed

## How to Deploy

### 1. Provision Infrastructure
Initialize and apply the Terraform configuration to build the VPC and EKS Cluster.

```bash
cd terraform
terraform init
terraform apply --auto-approve
```
### 2. Configure kubectl
Connect your local terminal to the new EKS cluster.
```bash
aws eks update-kubeconfig --region us-east-1 --name project-bedrock-cluster
```

### 3. Create the namespace
I noticed the project instructions mentioned a dist/kubernetes folder, but that folder doesn't exist in the current version of the repository (it looks like they moved to Helm charts recently). To make sure I was still deploying the correct app without changing the code myself, I used the official kubernetes.yaml release link provided by the repo maintainers instead.

kubectl create namespace retail-app
# Deploy the microservices using the Official Release Manifest
kubectl apply -f [https://github.com/aws-containers/retail-store-sample-app/releases/latest/download/kubernetes.yaml](https://github.com/aws-containers/retail-store-sample-app/releases/latest/download/kubernetes.yaml) -n retail-app

### 4. Change service type to LoadBalancer
Expose the UI service to the internet and get the Load Balancer URL.
kubectl patch service ui -n retail-app -p '{"spec": {"type": "LoadBalancer"}}'
# Get the URL (Copy the EXTERNAL-IP)
kubectl get svc ui -n retail-app

### 5. To avoid cloud costs, destroy the resources when finished:
kubectl delete namespace retail-app
cd terraform
terraform destroy --auto-approve
