# Production-Ready Web Infrastructure using AWS

##  Project Overview

Designed and implemented a production-ready web server infrastructure on AWS to provide high availability, scalability, and reliable traffic distribution.

The infrastructure includes a custom VPC with public and private subnets across three Availability Zones. Apache web servers are deployed on EC2 instances using a Launch Template and managed through an Auto Scaling Group. An Application Load Balancer distributes incoming traffic across the EC2 instances.

AWS Certificate Manager (ACM) and Amazon Route 53 are configured to provide secure HTTPS access through a custom domain.

---

##  Architecture

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

Target Group

Created a Target Group:

Name: web-tg

Configuration:

Target Type: Instances
VPC: prod-vpc
Healthy Threshold: 2
Unhealthy Threshold: 3
Health Check Interval: 30 seconds
📈 Auto Scaling Group

Created an Auto Scaling Group:

Name: web-asg

Configuration:

Launch Template: web-template
Desired Capacity: 3
Minimum Capacity: 1
Maximum Capacity: 7
Availability Zones: ap-south-1a, ap-south-1b, ap-south-1c
Target Group: web-tg

The Auto Scaling Group automatically launched three EC2 instances.

🔒 HTTPS Configuration

Requested a public SSL/TLS certificate using AWS Certificate Manager.

Certificate: *.poojadaingade.shop
DNS validation was configured using Route 53.
The certificate was successfully issued.
An HTTPS listener was added to the Application Load Balancer.
The ACM certificate was attached to the HTTPS listener.
🌍 Route 53 Configuration

Configured an A record in the Route 53 hosted zone:

Hosted Zone: poojadaingade.shop
Record Name: api
Record Type: A
Alias: Enabled
Endpoint: Application and Classic Load Balancer
Region: Asia Pacific (Mumbai)
Target: web-ALB

The configuration routes:

api.poojadaingade.shop → Application Load Balancer
🧪 Testing

The infrastructure was tested by accessing the Application Load Balancer DNS.

After refreshing the page, different EC2 server hostnames were displayed, verifying that traffic was being distributed across the EC2 instances.

The custom domain was also accessed successfully:

api.poojadaingade.shop

**Pooja Daingade**
