# Kubernetes Setup Guide

This guide walks you through setting up a Kubernetes cluster using Minikube, configuring a local Docker registry, managing secrets, setting up FluxCD, and deploying WordPress with Helm.

## Prerequisites
- OS: Linux Ubuntu x86-64
- Access to AWS for ECR
- GitHub repository access

## 1. Kubernetes Cluster with Minikube
Minikube is a tool that lets you run Kubernetes locally. To install and start Minikube, execute the following commands:

    curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
    sudo install minikube-linux-amd64 /usr/local/bin/minikube
    minikube start
    minikube status

For more detailed instructions, visit the [Minikube documentation](https://minikube.sigs.k8s.io/docs/start/).

## 2. Setup Local Docker Registry
To set up a local Docker registry, you need to create a public Docker registry on AWS ECR. Follow AWS's documentation for setting up an ECR registry.

## 3. Managing Kubernetes Secrets
Create a Kubernetes secret for MySQL. Note that this method is not recommended for production environments. Instead, consider integrating AWS Secrets Manager.

    kubectl create secret generic mysql-secret --from-literal=dbpassword=password --from-literal=rootpassword=password -n wordpress

## 4. FluxCD Local Setup
FluxCD automates the deployment of applications to Kubernetes. Install FluxCD and connect it to your GitHub repository with the following commands:

    curl -s https://fluxcd.io/install.sh | sudo bash
    flux bootstrap github --owner=<GitHub Username> --repository=<Repository Name> --branch=main --path=./clusters/my-cluster --personal --components-extra=image-reflector-controller,image-automation-controller

## 5. WordPress Deployment with Helm
A Helm Chart for WordPress is already prepared with the following structure:

    wordpress/
    ├── Chart.yaml
    ├── values.yaml
    └── templates/
        ├── wordpress-deployment.yaml
        ├── wordpress-pvc.yaml
        ├── wordpress-service.yaml
        ├── mysql-deployment.yaml
        ├── mysql-pvc.yaml
        └── mysql-service.yaml

## 6. Creating Namespace and Managing Secrets
Create a `wordpress` namespace and manage secrets for Git:

    kubectl create ns wordpress
    kubectl get secrets flux-system -n flux-system -o yaml > git-token.yaml
    # Modify the git-token.yaml file for the Wordpress namespace and apply it
    kubectl apply -f git-token.yaml

## 7. WordPress Configuration with Flux
Add the following files to the `wordpress/flux` folder for FluxCD to manage the deployment:

- `kustomization.yaml`
- `wordpress-helmrelease.yaml`
- `wordpress-imagepolicy.yaml`
- `wordpress-imagerepo.yaml`
- `wordpress-imageupdateautomation.yaml`

## 8. Finally, add the WordPress kustomization to Flux:

    flux create source git wordpress --url=<Repository Name> --branch=main --interval=10m
    flux create kustomization wordpress --source=flux-system --path="./clusters/wordpress/flux" --prune=true --interval=10m
    
---

Replace placeholders like `<GitHub Username>` and `<Repository Name>` with your actual GitHub username and repository name. Adjust any paths or configurations as needed for your environment.
