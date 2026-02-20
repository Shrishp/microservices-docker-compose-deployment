# microservices-docker-compose-deployment

Online Boutique – Docker Compose Deployment
📌 Project Overview

This project converts the Google Cloud Microservices Demo (Online Boutique) — originally designed for Kubernetes — into a Docker Compose-based architecture running on Ubuntu.

The goal was to:

Understand microservices architecture

Replace Kubernetes orchestration with Docker Compose

Configure service-to-service communication manually

Resolve cross-architecture (ARM vs AMD64) issues

Debug distributed container failures

🏗 Architecture

The application is a cloud-native e-commerce platform composed of multiple independent microservices communicating via gRPC.

🧩 Services Included

frontend

productcatalogservice

currencyservice

cartservice

recommendationservice

shippingservice

paymentservice

emailservice

checkoutservice

adservice

redis (database)

Each service runs in its own container.

🛠 Tech Stack

Ubuntu 24.04 LTS

Docker CE (Official Repository)

Docker Compose Plugin

Redis

gRPC-based microservices

QEMU (for cross-architecture emulation)

💻 Environment Details
Component	Value
Host Machine	MacBook Pro M2 Pro (ARM64)
VM	Ubuntu 24.04
Architecture	ARM64
Container Architecture	AMD64 (via emulation)
🚀 Installation & Setup
1️⃣ Install Docker (Official Method)

Remove old docker:

sudo apt remove docker.io -y

Install Docker official repository:

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

Verify:

docker --version
docker compose version
2️⃣ Enable Multi-Architecture Support (Important for M2 ARM)

Since the host machine is ARM64 but images are AMD64:

docker run --privileged --rm tonistiigi/binfmt --install all

Verify:

docker run --rm --platform linux/amd64 alpine uname -m

Expected output:

x86_64
3️⃣ Run the Application

Clone or create project folder:

mkdir online-boutique
cd online-boutique

Create docker-compose.yaml.

Then run:

docker compose up -d

Check status:

docker ps

All services should show:

Up
🌐 Access Application

Inside VM:

curl localhost:8080

From browser:

http://<VM-IP>:8080
🐞 Issues Faced & Solutions
❌ 1. Docker Compose Plugin Not Found

Cause:
Ubuntu default repo installs docker.io without compose plugin.

Solution:
Installed Docker from official Docker repository.

❌ 2. Unauthenticated Artifact Registry Error

Error:

denied: Unauthenticated request

Cause:
Google migrated images to Artifact Registry requiring authentication.

Solution:
Used public gcr.io images instead.

❌ 3. exec format error

Error:

exec /src/server: exec format error

Cause:
ARM64 host running AMD64 containers.

Solution:
Enabled multi-architecture emulation using QEMU and:

platform: linux/amd64
❌ 4. Frontend Restarting

Error:

panic: environment variable "AD_SERVICE_ADDR" not set

Cause:
Missing adservice container and environment variable.

Solution:
Added adservice and AD_SERVICE_ADDR=adservice:9555.
