## 🎯 Goal

EC2 in private subnet (no public IP, no IGW, no NAT)
Still able to connect using Session Manager via VPC Endpoint

Flow we are building:
EC2 → VPC Endpoint (private ENI) → AWS PrivateLink → SSM

## ✅ Prerequisites

You must have:
A VPC
Private subnet (example: 10.0.1.0/24)
EC2 inside that subnet
No public IP on EC2
IAM role ready: AmazonSSMManagedInstanceCore

## STEP 1 — Launch EC2 in private subnet

When launching EC2:
Subnet: Private subnet
Auto-assign public IP: ❌ Disabled
Security Group:
Outbound: allow all (default OK)
Inbound: nothing needed for SSM
Attach IAM role:
Attach role with policy:
✅ AmazonSSMManagedInstanceCore

## STEP 2 — Ensure SSM Agent is installed

Most AMIs already have it:
Amazon Linux 2/2023: already installed
Ubuntu: may need install
If needed (Ubuntu):
sudo snap install amazon-ssm-agent --classic
sudo systemctl enable amazon-ssm-agent
sudo systemctl start amazon-ssm-agent

## STEP 3 — Create VPC Endpoints (THIS IS THE MAGIC PART)

Go to:
AWS Console → VPC → Endpoints → Create endpoint
You must create 3 endpoints minimum.
Create them one by one:

Endpoint 1: SSM
Service name: com.amazonaws.ap-south-1.ssm
Type: Interface
VPC: your VPC
Subnets: select your private subnet
Enable: ✅ Private DNS
Security group: create or select one allowing:
Inbound: HTTPS (443) from your EC2 subnet CIDR (e.g., 10.0.1.0/24)
Create endpoint.

Endpoint 2: SSM Messages (required for Session Manager)
Service name: com.amazonaws.ap-south-1.ssmmessages
Same settings:
Interface endpoint
Same VPC
Same subnet
Private DNS enabled
Same SG (443 from private subnet)

Endpoint 3: EC2 Messages
Service name: com.amazonaws.ap-south-1.ec2messages
Same configuration.
(Optional but recommended)
Create S3 endpoint: com.amazonaws.ap-south-1.s3

This helps with:
Agent updates
Patch Manager
Run Command output

## STEP 4 — Fix security group of endpoint

Go to:
VPC → Endpoints → Select endpoint → Network Interface → Security Group

Inbound rule must be:
Type: HTTPS
Port: 443
Source: 10.0.1.0/24 (your private subnet CIDR)
Without this → SSM will fail.

## STEP 5 — Verify DNS behavior (optional but powerful check)

From EC2 (if you get shell access temporarily), run: nslookup ssm.ap-south-1.amazonaws.com

It should return:
👉 Private IP like 10.0.x.x
NOT public IP.

That confirms:
✅ Traffic is now private through endpoint

## STEP 6 — Go to Systems Manager → Session Manager

Now open:
AWS Console → Systems Manager → Session Manager → Start session

Your private EC2 should appear as:
Managed instance ✅
You can connect even though:
No public IP
No internet
No NAT
No IGW
