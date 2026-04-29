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
   - **Name:** `alb-sec-sg`
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
   - **Name:** `alb-tg`
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


## Step 5 — Create the Application Load Balancer

1. Go to **EC2 → Load Balancing → Load Balancers → Create Load Balancer**
2. Select **Application Load Balancer**
3. Configure:
   - **Name:** `alb1`
   - **Scheme:** Internet-facing
   - **IP address type:** IPv4
   - **VPC:** Default VPC
   - **Availability Zones:** Select **at least 2** (required by ALB)
   - **Security groups:** Remove default → add `alb-sec-sg`
4. Under **Listeners and routing:**
   - Protocol: HTTP | Port: 80
   - Default action: Forward to `alb-tg`
5. Click **Create Load Balancer**


## Step 6 — Verify ALB is Routing Traffic

1. Go to **Load Balancers → select `alb1`**
2. Copy the **DNS name** 
3. Go to **Target Groups → alb-tg → Targets tab**
4. Wait until both instances show **Healthy** status
5. Open `DNS name` in browser
6. Refresh several times — you should see **Server 1** and **Server 2** alternating

## Step 7 — Failover Test

1. Go to **EC2 → Instances → select `server-2`**
2. **Instance State → Stop Instance** → confirm
3. Wait ~60 seconds for the ALB health check to detect the instance is down
4. Refresh DNS page repeatedly
5. All traffic should now return **Server 2** only
6. To restore: **Start** `server-2` → wait ~60 seconds → both servers return






