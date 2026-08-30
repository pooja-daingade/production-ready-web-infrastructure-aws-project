# Production-Ready Web Infrastructure using AWS

##  Project Overview

Designed and implemented a production-ready web server infrastructure on AWS to provide high availability, scalability, and reliable traffic distribution.

The infrastructure uses a custom VPC with public and private subnets across three Availability Zones. Internet Gateway and NAT Gateways are configured for network connectivity. Apache web servers are deployed on EC2 instances using a Launch Template and managed through an Auto Scaling Group.

An Application Load Balancer distributes incoming traffic across the EC2 instances, while AWS Certificate Manager (ACM) and Amazon Route 53 are configured to provide HTTPS access through a custom domain.

---

##  Architecture

<img width="1536" height="1024" alt="architecture-diagram" src="https://github.com/user-attachments/assets/9fbb75e2-6534-4405-954e-95c03455922a" />


### Architecture Components

* Amazon VPC
* Public and Private Subnets
* Availability Zones
* Internet Gateway
* NAT Gateways
* Route Tables
* Security Groups
* Amazon EC2
* Launch Template
* Auto Scaling Group
* Application Load Balancer
* Target Group
* AWS Certificate Manager (ACM)
* Amazon Route 53
* Apache Web Server

---

##  AWS Services Used

| AWS Service                   | Purpose                                              |
| ----------------------------- | ---------------------------------------------------- |
| **Amazon VPC**                | Created an isolated network environment              |
| **Amazon EC2**                | Hosts Apache web servers                             |
| **Application Load Balancer** | Distributes incoming traffic across EC2 instances    |
| **Auto Scaling Group**        | Automatically manages EC2 instances                  |
| **Launch Template**           | Defines the EC2 instance configuration               |
| **Target Group**              | Registers and monitors EC2 instances                 |
| **Internet Gateway**          | Provides internet connectivity for public resources  |
| **NAT Gateway**               | Provides outbound connectivity for private resources |
| **Route Tables**              | Controls network traffic routing                     |
| **Security Groups**           | Controls inbound traffic                             |
| **AWS Certificate Manager**   | Provides SSL/TLS certificate                         |
| **Amazon Route 53**           | Configures DNS and routes the domain to the ALB      |

---

##  Network Configuration

### VPC

* **VPC Name:** `prod-vpc`
* **CIDR:** `10.0.0.0/16`
* **Region:** Asia Pacific (Mumbai)

### Public Subnets

| Subnet                 | Availability Zone | CIDR          |
| ---------------------- | ----------------- | ------------- |
| `public-1-ap-south-1a` | ap-south-1a       | `10.0.1.0/24` |
| `public-2-ap-south-1b` | ap-south-1b       | `10.0.2.0/24` |
| `public-3-ap-south-1c` | ap-south-1c       | `10.0.3.0/24` |

### Private Subnets

| Subnet                  | Availability Zone | CIDR           |
| ----------------------- | ----------------- | -------------- |
| `private-1-ap-south-1a` | ap-south-1a       | `10.0.11.0/24` |
| `private-2-ap-south-1b` | ap-south-1b       | `10.0.12.0/24` |
| `private-3-ap-south-1c` | ap-south-1c       | `10.0.13.0/24` |

---

##  Internet Gateway

Created and attached an Internet Gateway:

**Name:** `my-prod-igw`

The public route table uses:

```text
0.0.0.0/0 → Internet Gateway
```

The public subnets are associated with the public route table.

---

##  NAT Gateways

Three zonal NAT Gateways were created, one in each public subnet.

| NAT Gateway          | Subnet               | Availability Zone |
| -------------------- | -------------------- | ----------------- |
| `my-nat-ap-south-1a` | public-1-ap-south-1a | ap-south-1a       |
| `my-nat-ap-south-1b` | public-2-ap-south-1b | ap-south-1b       |
| `my-nat-ap-south-1c` | public-3-ap-south-1c | ap-south-1c       |

Each NAT Gateway was configured with an Elastic IP.

---

##  Security Groups

### ALB Security Group

Allows:

* HTTP — Port 80
* HTTPS — Port 443
* Source: `0.0.0.0/0`

### EC2 Security Group

Allows:

* HTTP — Port 80
* HTTPS — Port 443
* Source: `0.0.0.0/0`

---

##  EC2 Launch Template

Created a Launch Template named:

**`web-template`**

Configuration:

* **AMI:** Ubuntu
* **Instance Type:** `t3.micro`
* **Version:** v1
* **Availability Zone:** ap-south-1a
* **Security Group:** EC2 Security Group

### User Data

The User Data automatically:

1. Updates the Ubuntu packages
2. Installs Apache
3. Enables Apache
4. Starts Apache
5. Displays the EC2 hostname on the web page

```bash
#!/bin/bash
apt update -y
apt install apache2 -y
systemctl enable apache2
systemctl start apache2
echo "Server: $(hostname)" > /var/www/html/index.html
```

---

##  Application Load Balancer

Created an internet-facing Application Load Balancer:

**Name:** `web-ALB`

Configuration:

* **Type:** Application Load Balancer
* **Scheme:** Internet-facing
* **VPC:** `prod-vpc`
* **Availability Zones:** ap-south-1a, ap-south-1b, ap-south-1c
* **Security Group:** ALB Security Group
* **Target Group:** `web-tg`

The ALB distributes incoming requests across the EC2 instances.

---

##  Target Group

Created a Target Group:

**Name:** `web-tg`

Configuration:

* **Target Type:** Instances
* **VPC:** `prod-vpc`
* **Healthy Threshold:** 2
* **Unhealthy Threshold:** 3
* **Health Check Interval:** 30 seconds

---

##  Auto Scaling Group

Created an Auto Scaling Group:

**Name:** `web-asg`

Configuration:

* **Launch Template:** `web-template`
* **Desired Capacity:** 3
* **Minimum Capacity:** 1
* **Maximum Capacity:** 7
* **Availability Zones:** ap-south-1a, ap-south-1b, ap-south-1c
* **Target Group:** `web-tg`

The Auto Scaling Group automatically launched three EC2 instances.

---

##  HTTPS Configuration

Requested a public SSL/TLS certificate using AWS Certificate Manager.

* **Certificate:** `*.poojadaingade.shop`
* DNS validation was configured using Route 53.
* The certificate was successfully issued.
* An HTTPS listener was added to the Application Load Balancer.
* The ACM certificate was attached to the HTTPS listener.

---

##  Route 53 Configuration

Configured an A record in the Route 53 hosted zone:

* **Hosted Zone:** `poojadaingade.shop`
* **Record Name:** `api`
* **Record Type:** A
* **Alias:** Enabled
* **Endpoint:** Application and Classic Load Balancer
* **Region:** Asia Pacific (Mumbai)
* **Target:** `web-ALB`

The configuration routes:

```text
api.poojadaingade.shop → Application Load Balancer
```

---

##  Testing

The infrastructure was tested by accessing the Application Load Balancer DNS.

After refreshing the page, different EC2 server hostnames were displayed, verifying that traffic was being distributed across the EC2 instances.

The custom domain was also accessed successfully:

```text
api.poojadaingade.shop
```

---

##  Key Features

* Multi-AZ network architecture
* Public and private subnet configuration
* Internet-facing Application Load Balancer
* EC2-based Apache web servers
* Auto Scaling for EC2 instances
* Health checks through Target Group
* NAT Gateway configuration
* HTTPS using ACM
* DNS routing using Route 53
* Load distribution across multiple EC2 instances

---

```

---

##  Author

**Pooja Daingade**
