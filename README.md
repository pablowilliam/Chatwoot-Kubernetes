# Chatwoot-Kubernetes
Create a Chatwoot instance with 5 Repilcas using Dookcer and minicube for study.

# Step-by-Step Guide

Step 1: Install Docker
```bash
# Update the package list

sudo apt update

# Install Docker
sudo apt install -y docker.io

# Enable and start Docker
sudo systemctl enable docker
sudo systemctl start docker

# Add current user to the Docker group
sudo usermod -aG docker $USER

