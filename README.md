# SonarQube Deployment on AWS using Docker, Private EC2, Bastion Host and Application Load Balancer

## Overview

This project demonstrates how to deploy **SonarQube** in a **production-style AWS architecture** using the following principles:

* Services run inside **private subnets**
* Internet access is provided through an **Application Load Balancer**
* Administration access is controlled via a **Bastion host**
* The application runs inside **Docker containers**

The setup simulates a **real DevOps infrastructure pattern** used in production environments where applications are isolated from the public internet while still being accessible through a load balancer.

This project is useful for learning:

* AWS networking (VPC, subnets, routing)
* Application Load Balancer configuration
* Docker container deployment
* Secure server access using Bastion hosts
* Deploying SonarQube for CI/CD pipelines

---

# Architecture

```
Internet
   │
   ▼
Application Load Balancer (Public Subnets)
   │
   ▼
Target Group (HTTP : 9000)
   │
   ▼
EC2 Instance (Private Subnet)
Docker Container (SonarQube :9000)
   │
   ▼
Bastion Host (Public Subnet for SSH)
```

### Architecture Principles

* Public access is **only allowed through ALB**
* Application servers remain **isolated in private subnets**
* SSH access is restricted through a **Bastion host**
* Docker containers provide **portable application deployment**

---

# AWS Infrastructure Components

## 1. VPC

A custom Virtual Private Cloud is created to isolate the environment.

```
CIDR Block: 10.0.0.0/16
```

The VPC allows us to control networking, routing and security.

---

## 2. Subnets

Three subnets are used in this architecture.

| Subnet           | Type    | Purpose                   |
| ---------------- | ------- | ------------------------- |
| Public Subnet 1  | Public  | Bastion Host              |
| Public Subnet 2  | Public  | Application Load Balancer |
| Private Subnet 1 | Private | SonarQube Server          |

### Public Subnets

Public subnets have a route to the Internet Gateway.

```
0.0.0.0/0 → Internet Gateway
```

Resources placed here can communicate with the internet.

### Private Subnets

Private subnets **do not have direct internet access**.

Application servers like SonarQube are placed here for security.

---

## 3. Internet Gateway

An **Internet Gateway (IGW)** is attached to the VPC to allow internet connectivity for public subnets.

---

# EC2 Instances

## Bastion Host

The Bastion host is used to securely SSH into private instances.

| Property      | Value              |
| ------------- | ------------------ |
| Instance Type | t3.micro           |
| Subnet        | Public             |
| OS            | Ubuntu             |
| Purpose       | Secure SSH gateway |

### Connect to Bastion

```
ssh -i key.pem ubuntu@<BASTION_PUBLIC_IP>
```

---

## SonarQube Server

The SonarQube server runs inside a **private subnet** and hosts the application container.

| Property      | Value            |
| ------------- | ---------------- |
| Instance Type | m7i-flex.large   |
| Subnet        | Private          |
| OS            | Ubuntu 24.04     |
| Application   | Docker container |

The server **does not have a public IP address**.

---

# Security Groups

## ALB Security Group

Allows HTTP access from the internet.

| Type | Port | Source    |
| ---- | ---- | --------- |
| HTTP | 80   | 0.0.0.0/0 |

---

## Bastion Host Security Group

Allows SSH access from the developer machine.

| Type | Port | Source |
| ---- | ---- | ------ |
| SSH  | 22   | My IP  |

---

## SonarQube EC2 Security Group

Allows application traffic from the load balancer.

| Type | Port | Source                 |
| ---- | ---- | ---------------------- |
| SSH  | 22   | Bastion Security Group |
| TCP  | 9000 | ALB Security Group     |

---

# Application Load Balancer (ALB)

The Application Load Balancer exposes the application to the internet.

### Configuration

| Setting  | Value                     |
| -------- | ------------------------- |
| Type     | Application Load Balancer |
| Scheme   | Internet-facing           |
| Listener | HTTP : 80                 |
| Subnets  | Public Subnets            |

---

# Target Group

The ALB routes traffic to the target group which forwards requests to the application server.

### Target Group Configuration

| Setting           | Value    |
| ----------------- | -------- |
| Target Type       | Instance |
| Protocol          | HTTP     |
| Port              | 9000     |
| Health Check Path | /        |
| Success Codes     | 200-399  |

---

# Listener Rule

```
Listener: HTTP 80
Forward → Target Group (Port 9000)
```

---

# Connecting to the Private Server

Step 1 — Connect to Bastion

```
ssh -i bastion-key.pem ubuntu@<BASTION_PUBLIC_IP>
```

Step 2 — Connect to Private Server

```
ssh -i server-key.pem ubuntu@10.0.x.x
```

---

# Installing Docker

Update system packages

```
sudo apt update
```

Install Docker

```
sudo apt install docker.io -y
```

Start Docker

```
sudo systemctl start docker
sudo systemctl enable docker
```

Add user to Docker group

```
sudo usermod -aG docker ubuntu
newgrp docker
```

---

# Running SonarQube Container

Pull and run SonarQube container

```
docker run -d -p 9000:9000 sonarqube:lts
```

Check container status

```
docker ps
```

---

# Verify Application Locally

```
curl localhost:9000
```

---

# Verify from Bastion

```
curl http://10.0.x.x:9000
```

---

# Access Application from Browser

Open the ALB DNS:

```
http://<ALB-DNS>
```

Example:

```
http://alb-3-593350159.us-east-2.elb.amazonaws.com
```

---

# Default SonarQube Login

```
username: admin
password: admin
```

---

# Troubleshooting

## 503 Service Unavailable

Possible causes:

* Target group port mismatch
* Incorrect security group rules
* Health checks failing
* Container not running

---

## Docker Permission Error

Error:

```
permission denied while trying to connect to docker.sock
```

Fix:

```
sudo usermod -aG docker ubuntu
newgrp docker
```

---

# Final Result

The SonarQube application is now securely deployed using:

* Docker
* Private EC2 instance
* Bastion host
* Application Load Balancer
* AWS VPC networking

This architecture follows **production-grade DevOps security practices**.

---

