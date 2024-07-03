# Chatwoot-Kubernetes
Chatwoot gives you all the tools to manage conversations, build relationships and delight your customers from one place.

# Step-by-Step Guide

  # Step 1: Install Docker

''bash
# Update the package list
sudo apt update

# Install Docker
sudo apt install -y docker.io

# Enable and start Docker
sudo systemctl enable docker
sudo systemctl start docker

# Add current user to the Docker group
sudo usermod -aG docker $USER
''bash
