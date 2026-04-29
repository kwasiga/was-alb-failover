# EC2 + ALB Load Balancing Lab

## Overview
Two EC2 instances behind an Application Load Balancer (ALB).
Demonstrates traffic distribution and single-instance failover.

## Step 1 — Launch EC2 Instances
1. Go to **EC2 → Instances → Launch Instance**
2. Configure:
   - **Name:** `server-1`
   - **AMI:** Amazon Linux 2023 (free tier eligible)
   - **Instance type:** `t3.micro`
   - **Key pair:** Create new → download `.pem` file (keep it safe)
3. Expand **Advanced Details → User Data**, paste:
 ```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Server [#number]</h1>" > /var/www/html/index.html
```
4. **Launch Instance**
## Step 2 —Verify Both Servers
2. Copy the **Public IPv4 address** of each instance
3. Open `http://<instance-1-public-ip>` in browser → should show **Server [num]**
4. Open `http://<instance-2-public-ip>` in browser → should show **Server [num]**


## Step 3 — Create ALB Security Group

Before creating the ALB, create a dedicated security group for it.

1. Go to **EC2 → Network & Security → Security Groups → Create Security Group**
2. Configure:
   - **Name:** `alb-1`
   - **Description:** ALB public traffic
   - **VPC:** Default VPC
   - **Inbound rules:**
     - HTTP | Port 80 | Source: Anywhere (0.0.0.0/0)
   - **Outbound rules:** Leave default (all traffic)
3. **Create Security Group**


## Step 4 — Create Target Group

1. Go to **EC2 → Load Balancing → Target Groups → Create Target Group**
2. Configure:
   - **Target type:** Instances
   - **Name:** `ec2-target-group`
   - **Protocol:** HTTP
   - **Port:** 80
   - **VPC:** Default VPC
   - **Health check protocol:** HTTP
   - **Health check path:** `/`
3. Click **Next**
4. Under **Register Targets:**
   - Select both `server-1` and `server-2`
   - Click **Include as pending below**
5. Click **Create Target Group**






