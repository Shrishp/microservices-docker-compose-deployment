# Online Boutique – Docker Compose Deployment

## Project Overview

This project converts the Google Cloud **Microservices Demo (Online Boutique)** — originally designed for Kubernetes — into a Docker Compose-based deployment running on Ubuntu.

The objective was to:

- Replace Kubernetes orchestration with Docker Compose
- Manually configure service-to-service communication
- Run a distributed microservices architecture locally
- Resolve ARM64 (Apple Silicon) vs AMD64 container compatibility
- Debug multi-container dependency failures

---

# Architecture

Online Boutique is a cloud-native e-commerce platform composed of multiple independent microservices communicating using gRPC.

Each service runs in its own container and communicates through Docker’s internal bridge network.

---

## Services Included

- frontend
- productcatalogservice
- currencyservice
- cartservice
- recommendationservice
- shippingservice
- paymentservice
- emailservice
- checkoutservice
- adservice
- redis

---

# Tech Stack

- Ubuntu 24.04 LTS
- Docker CE (Official Repository)
- Docker Compose Plugin
- Redis
- gRPC-based microservices
- QEMU (Multi-Architecture Emulation)

---

# Environment Details

| Component | Value |
|-----------|--------|
| Host Machine | MacBook Pro M2 Pro |
| Architecture | ARM64 |
| VM | Ubuntu 24.04 |
| Container Architecture | AMD64 (via emulation) |

---

# Installation & Setup

## 1️⃣ Install Docker (Official Method)

Remove old Docker if installed:

```bash
sudo apt remove docker.io -y
sudo apt autoremove -y
```

Install Docker Official Repository:

```bash
sudo apt update
sudo apt install ca-certificates curl gnupg -y

sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo tee /etc/apt/keyrings/docker.asc > /dev/null
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu noble stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

Verify:

```bash
docker --version
docker compose version
```

---

## 2️⃣ Enable Multi-Architecture Support (For Apple Silicon / ARM)

Since the host machine is ARM64 but images are AMD64, enable QEMU emulation:

```bash
docker run --privileged --rm tonistiigi/binfmt --install all
```

Verify:

```bash
docker run --rm --platform linux/amd64 alpine uname -m
```

Expected Output:

```
x86_64
```

---

## 3️⃣ Project Setup

Create project directory:

```bash
mkdir online-boutique
cd online-boutique
```

Create a file:

```
docker-compose.yaml
```

---

# docker-compose.yaml

```yaml
services:

  redis:
    image: redis:7.2-alpine
    restart: always

  productcatalogservice:
    image: gcr.io/google-samples/microservices-demo/productcatalogservice:v0.8.0
    platform: linux/amd64
    environment:
      - PORT=3550
    restart: always

  currencyservice:
    image: gcr.io/google-samples/microservices-demo/currencyservice:v0.8.0
    platform: linux/amd64
    environment:
      - PORT=7000
    restart: always

  cartservice:
    image: gcr.io/google-samples/microservices-demo/cartservice:v0.8.0
    platform: linux/amd64
    environment:
      - REDIS_ADDR=redis:6379
      - PORT=7070
    restart: always

  recommendationservice:
    image: gcr.io/google-samples/microservices-demo/recommendationservice:v0.8.0
    platform: linux/amd64
    environment:
      - PORT=8080
      - PRODUCT_CATALOG_SERVICE_ADDR=productcatalogservice:3550
    restart: always

  shippingservice:
    image: gcr.io/google-samples/microservices-demo/shippingservice:v0.8.0
    platform: linux/amd64
    environment:
      - PORT=50051
    restart: always

  paymentservice:
    image: gcr.io/google-samples/microservices-demo/paymentservice:v0.8.0
    platform: linux/amd64
    environment:
      - PORT=50051
    restart: always

  emailservice:
    image: gcr.io/google-samples/microservices-demo/emailservice:v0.8.0
    platform: linux/amd64
    environment:
      - PORT=8080
    restart: always

  checkoutservice:
    image: gcr.io/google-samples/microservices-demo/checkoutservice:v0.8.0
    platform: linux/amd64
    environment:
      - PORT=5050
      - PRODUCT_CATALOG_SERVICE_ADDR=productcatalogservice:3550
      - SHIPPING_SERVICE_ADDR=shippingservice:50051
      - PAYMENT_SERVICE_ADDR=paymentservice:50051
      - EMAIL_SERVICE_ADDR=emailservice:8080
      - CURRENCY_SERVICE_ADDR=currencyservice:7000
      - CART_SERVICE_ADDR=cartservice:7070
    restart: always

  adservice:
    image: gcr.io/google-samples/microservices-demo/adservice:v0.8.0
    platform: linux/amd64
    environment:
      - PORT=9555
    restart: always

  frontend:
    image: gcr.io/google-samples/microservices-demo/frontend:v0.8.0
    platform: linux/amd64
    ports:
      - "8080:8080"
    environment:
      - PORT=8080
      - PRODUCT_CATALOG_SERVICE_ADDR=productcatalogservice:3550
      - CURRENCY_SERVICE_ADDR=currencyservice:7000
      - CART_SERVICE_ADDR=cartservice:7070
      - RECOMMENDATION_SERVICE_ADDR=recommendationservice:8080
      - SHIPPING_SERVICE_ADDR=shippingservice:50051
      - CHECKOUT_SERVICE_ADDR=checkoutservice:5050
      - AD_SERVICE_ADDR=adservice:9555
    restart: always
```

---

# ▶ Run Application

```bash
docker compose up -d
```

Check status:

```bash
docker ps
```

All containers should show:

```
Up
```

---

# Access Application

Inside VM:

```bash
curl localhost:8080
```

From browser:

```
http://<VM-IP>:8080
```

---

# Issues Faced & Resolved

### 1. Docker Compose Plugin Not Found
Cause: Ubuntu default `docker.io` does not include compose plugin  
Solution: Installed Docker from official Docker repository

### 2. Artifact Registry Authentication Error
Cause: Google moved images to Artifact Registry requiring authentication  
Solution: Used public `gcr.io` images

### 3. exec format error
Cause: ARM64 host running AMD64 containers  
Solution: Enabled multi-architecture support using QEMU

### 4. Frontend Crash Loop
Cause: Missing `AD_SERVICE_ADDR` environment variable  
Solution: Added `adservice` and required environment variable

---

# DevOps Concepts Demonstrated

- Microservices architecture
- Docker networking & service discovery
- Container debugging & log analysis
- Cross-architecture compatibility (ARM vs AMD)
- Multi-container orchestration
- Distributed system troubleshooting
- Production-style debugging approach

---

# Final Outcome

Successfully deployed a distributed microservices architecture using Docker Compose on an ARM-based system with AMD64 container emulation.
