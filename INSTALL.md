# Minikube Installation Guide

This guide walks you through the process of installing Minikube on Ubuntu. Follow the steps below to set up Minikube and start a Kubernetes cluster locally.

## 1. Install Dependencies

```bash
sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release
```

## 2. Install Docker

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo groupadd docker
sudo usermod -aG docker $USER # or specify the user instead of $USER
newgrp docker # or restart the session
docker --version
```

## 3. Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
```

## 4. Install Minikube

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
minikube version
```

## 5. Run Minikube

```bash
minikube start --driver=docker 
kubectl config view
```

## 6. Enable kubectl Auto-Completion

```bash
apt-get install bash-completion 
kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl > /dev/null
# Reload shell
```

## Expose Pod Port in Minikube

```bash
kubectl create deployment hello-node --image=k82.gcr.io/echoserver:1.4
kubectl expose deployment hello-node --type=LoadBalancer --port=8080
minikube service hello-node
```

Follow these steps to set up Minikube and start using Kubernetes locally. Enjoy experimenting with Kubernetes!