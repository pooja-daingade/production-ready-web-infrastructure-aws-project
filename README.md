# Production-Ready Web Infrastructure using AWS

##  Project Overview

Designed and implemented a production-ready web server infrastructure on AWS to provide high availability, scalability, and reliable traffic distribution.

The infrastructure includes a custom VPC with public and private subnets across three Availability Zones. Apache web servers are deployed on EC2 instances using a Launch Template and managed through an Auto Scaling Group. An Application Load Balancer distributes incoming traffic across the EC2 instances.

AWS Certificate Manager (ACM) and Amazon Route 53 are configured to provide secure HTTPS access through a custom domain.

---

## 🏗️ Architecture

<img width="1536" height="1024" alt="architecture-diagram" src="https://github.com/user-attachments/assets/6a0a018c-7714-447a-a35c-69c3443047d6" />


### Architecture Components

- Amazon VPC
- Public and Private Subnets
- Internet Gateway
- NAT Gateways
- Route Tables
- Security Groups
- Amazon EC2
- Launch Template
- Application Load Balancer (ALB)
- Target Group
- Auto Scaling Group
- AWS Certificate Manager (ACM)
- Amazon Route 53

---

##  AWS Configuration

### VPC

- VPC Name: `prod-vpc`
- CIDR: `10.0.0.0/16`
- Region: `ap-south-1 (Mumbai)`

### Public Subnets

| Subnet | Availability Zone | CIDR |
|---|---|---|
| public-1-ap-south-1a | ap-south-1a | 10.0.1.0/24 |
| public-2-ap-south-1b | ap-south-1b | 10.0.2.0/24 |
| public-3-ap-south-1c | ap-south-1c | 10.0.3.0/24 |

### Private Subnets

| Subnet | Availability Zone | CIDR |
|---|---|---|
| private-1-ap-south-1a | ap-south-1a | 10.0.11.0/24 |
| private-2-ap-south-1b | ap-south-1b | 10.0.12.0/24 |
| private-3-ap-south-1c | ap-south-1c | 10.0.13.0/24 |

---

##  Network Configuration

### Internet Gateway

An Internet Gateway named `my-prod-igw` was created and attached to the VPC.

### NAT Gateways

Three zonal NAT Gateways were created, one in each public subnet, with Elastic IP addresses.

- `my-nat-ap-south-1a`
- `my-nat-ap-south-1b`
- `my-nat-ap-south-1c`

### Route Tables

A public route table was configured for the public subnets.

Private route tables were created separately for each Availability Zone.

---

##  Security Groups

### ALB Security Group

Allows:

- HTTP (80)
- HTTPS (443)

from `0.0.0.0/0`.

### EC2 Security Group

Configured to allow:

- HTTP (80)
- HTTPS (443)

---

##  EC2 Web Server

A Launch Template named `web-template` was created with:

- AMI: Ubuntu
- Instance Type: `t3.micro`
- Version: `v1`

Apache was automatically installed and started using User Data.

```bash
#!/bin/bash
apt update -y
apt install apache2 -y
systemctl enable apache2
systemctl start apache2
echo "Server: $(hostname)" > /var/www/html/index.html

##  Target Group

A Target Group named `web-tg` was created to register and manage the EC2 instances.

* Target Type: Instances
* Target Group Name: `web-tg`
* VPC: `prod-vpc`
* Healthy Threshold: 2
* Unhealthy Threshold: 3
* Health Check Interval: 30 seconds

---

##  Application Load Balancer

An internet-facing Application Load Balancer named `web-ALB` was created across three Availability Zones.

* Type: Application Load Balancer
* Name: `web-ALB`
* Scheme: Internet-facing
* VPC: `prod-vpc`
* Security Group: ALB Security Group
* Target Group: `web-tg`

### Availability Zones

* `ap-south-1a` → `public-1-ap-south-1a`
* `ap-south-1b` → `public-2-ap-south-1b`
* `ap-south-1c` → `public-3-ap-south-1c`

---

##  Auto Scaling Group

An Auto Scaling Group named `web-asg` was created using the `web-template`.

| Configuration      | Value          |
| ------------------ | -------------- |
| Auto Scaling Group | `web-asg`      |
| Launch Template    | `web-template` |
| Desired Capacity   | 3              |
| Minimum Capacity   | 1              |
| Maximum Capacity   | 7              |

The Auto Scaling Group uses the three private subnets across the Availability Zones.

Three EC2 instances were automatically launched by the Auto Scaling Group.

---

##  HTTPS Configuration

A public SSL/TLS certificate was requested using AWS Certificate Manager (ACM).

* Domain: `*.poojadaingade.shop`

DNS validation was performed using a CNAME record in Route 53.

An HTTPS listener was then configured on the Application Load Balancer using the ACM certificate.

---

##  Route 53

A Route 53 A-record Alias was configured to route the custom domain to the Application Load Balancer.

* Record Name: `api`
* Record Type: A
* Alias: On
* Region: Asia Pacific (Mumbai)
* Target: `web-ALB`

The configured domain was:

`api.poojadaingade.shop`

---

##  Testing & Verification

The Application Load Balancer DNS was accessed to verify the web server.

On refreshing the page, different EC2 server hostnames were displayed, verifying that traffic was being distributed across the EC2 instances.

The custom domain was also accessed successfully and the server response was verified.

---

##  Key Features

* Multi-AZ AWS infrastructure
* Custom VPC with public and private subnets
* Internet Gateway and NAT Gateways
* EC2 Apache web servers
* Launch Template
* Application Load Balancer
* Target Group with health checks
* Auto Scaling Group
* HTTPS using AWS Certificate Manager
* Custom domain routing using Route 53
* Traffic distribution across EC2 instances

---

##  Project Documentation

Detailed implementation steps and screenshots are available in:

**Production-Ready-Web-Infrastructure-using-AWS.pdf**

---

##  Author

**Pooja Daingade**
