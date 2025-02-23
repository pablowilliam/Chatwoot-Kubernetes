# Chatwoot-Kubernetes
Create a Chatwoot instance with 5 Repilcas using Dookcer and minicube for study.

# Step-by-Step Guide for Installation and Configuration


Step 1: Update and Install Dependencies on Ubuntu Server 20.04
```bash
# Update the package list and install dependencies
sudo apt update
sudo apt install -y docker.io curl apt-transport-https gnupg lsb-release

```

# Step 2: Install Docker
```bash
# Install Docker
sudo apt install -y docker.io

# Start and enable Docker
sudo systemctl start docker
sudo systemctl enable docker

# Add the current user to the Docker group
sudo usermod -aG docker $USER

# Restart session to apply group changes
newgrp docker

```

# Step 3: Install Minikube

```bash

# Download Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

# Install Minikube
sudo install minikube-linux-amd64 /usr/local/bin/minikube

```
# Step 4: Install Kubectl

```bash

# Download Kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Install Kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

```
# Step 5: Start Minikube

```bash

# Start Minikube with Docker driver
minikube start --driver=docker

```

# Step 6: Install Helm

```bash

# Download Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

```

# Step 7: Deploy Nginx Ingress Controller

```bash

# Add the ingress-nginx repository
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

# Update the helm repository
helm repo update

# Install the ingress-nginx controller
helm install ingress-nginx ingress-nginx/ingress-nginx
```

# Step 8: Create Kubernetes Deployment and Service for Chatwoot
Create a file named chatwoot-deployment.yaml with the following content:

```bash

apiVersion: apps/v1
kind: Deployment
metadata:
  name: chatwoot
  labels:
    app: chatwoot
spec:
  replicas: 1
  selector:
    matchLabels:
      app: chatwoot
  template:
    metadata:
      labels:
        app: chatwoot
    spec:
      containers:
      - name: chatwoot
        image: sendingtk/chatwoot:v3.13.8
        ports:
        - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: chatwoot-service
spec:
  selector:
    app: chatwoot
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
  type: NodePort

´´´

```
Apply the ingress:

```bash
kubectl apply -f chatwoot-ingress.yaml
```

# Step 9: Create Ingress Resource
Create a file named chatwoot-ingress.yaml with the following content:

```bash

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: chatwoot-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  rules:
  - host: yourdomain.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: chatwoot-service
            port:
              number: 80
  tls:
  - hosts:
    - yourdomain.example.com
    secretName: chatwoot-tls
```

Apply the ingress:

```bash

kubectl apply -f chatwoot-ingress.yaml

```

# Step 10: Install Cert-Manager for SSL

```bash
# Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io

```bash
# Update the helm repository
helm repo update

# Install Cert-Manager
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --version v1.3.1 --set installCRDs=true

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```
# Step 11: Create Cluster Issuer
Create a file named cluster-issuer.yaml with the following content:

```bash

apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
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

Apply the Cluster Issuer:

```bash

kubectl apply -f cluster-issuer.yaml

```

# Final Script
Save the following script as deploy_chatwoot.sh and run it:

```bash

#!/bin/bash

# Update the package list and install dependencies
sudo apt update
sudo apt install -y docker.io curl apt-transport-https gnupg lsb-release

# Install Docker
sudo apt install -y docker.io

# Start and enable Docker
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER
newgrp docker

# Install Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Install Kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Start Minikube
minikube start --driver=docker

# Install Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Deploy Nginx Ingress Controller
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx

# Create Chatwoot Deployment and Service
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: chatwoot
  labels:
    app: chatwoot
spec:
  replicas: 5
  selector:
    matchLabels:
      app: chatwoot
  template:
    metadata:
      labels:
        app: chatwoot
    spec:
      containers:
      - name: chatwoot
        image: chatwoot/chatwoot:latest
        ports:
        - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: chatwoot-service
spec:
  selector:
    app: chatwoot
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
  type: NodePort
EOF

# Create Chatwoot Ingress Resource
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: chatwoot-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  rules:
  - host: yourdomain.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: chatwoot-service
            port:
              number: 80
  tls:
  - hosts:
    - yourdomain.example.com
    secretName: chatwoot-tls
EOF

# Install Cert-Manager for SSL
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --version v1.3.1 --set installCRDs=true

# Create Cluster Issuer for Let's Encrypt
cat <<EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
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
EOF

```

Replace your-email@example.com and yourdomain.example.com with your actual email address and domain name, respectively. Save the script and make it executable:

```bash

chmod +x deploy_chatwoot.sh

```
Run the script:

```bash

./deploy_chatwoot.sh

```

This script installs Docker, Minikube, Kubectl, Helm, deploys Chatwoot on Kubernetes with 5 replicas, sets up an Nginx Ingress Controller, installs Cert-Manager for SSL, and config.



## Apoie este projeto ☕  

Se quiser contribuir, você pode me apoiar no [Buy Me a Coffee](https://www.buymeacoffee.com/pablowilliam) ou via PIX:  

**PIX:** contato@site.multichannel.pt  
