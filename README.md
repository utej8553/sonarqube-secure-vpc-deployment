# SonarQube Deployment on AWS (Secure VPC Architecture)

---

## Architecture

```
Internet
   │
   ▼
Application Load Balancer (Public Subnets)
   │
   ▼
SonarQube Server (Private Subnet)
   │
   ▼
NAT Gateway
   │
   ▼
Internet Gateway
```

Administrative access:

```
Developer Machine
      │
      ▼
Bastion Host (Public Subnet)
      │
      ▼
SonarQube Server (Private Subnet)
```

## Deployment

1. Created a **custom VPC** with public and private subnets.
2. Configured an **Internet Gateway** for public access.
3. Added a **NAT Gateway** to allow private instances outbound internet access.
4. Launched a **bastion host** in the public subnet for secure SSH access.
5. Deployed the **SonarQube EC2 instance** inside the private subnet.
6. Installed **Docker** and ran SonarQube using a container.
7. Created an **Application Load Balancer (ALB)** to route HTTP traffic to the SonarQube server.
8. Configured **security groups** to restrict traffic between components.

---

## Running SonarQube

SonarQube runs in Docker on the private EC2 instance:

```bash
sudo docker run -d --name sonarqube -p 9000:9000 sonarqube:lts
```

---

## Accessing SonarQube

SonarQube is accessed through the Application Load Balancer:

```
http://<ALB-DNS>
```
