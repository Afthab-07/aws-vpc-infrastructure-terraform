# AWS VPC Project - Version 1 & Version 2

## Overview

This project shows how I learned to build AWS cloud systems step by step:
- **Version 1**: Basic network setup with load balancer and web servers
- **Version 2**: Improved version with cost-saving Spot instances and Auto Scaling

## Version 1: Simple AWS Setup

### What I Built

A simple website that runs on 2 servers in AWS behind a load balancer.

### AWS Services Used
- **VPC**: Network container for everything
- **Subnets**: 2 public subnets in different areas (high availability)
- **Internet Gateway**: Lets internet traffic in and out
- **ALB (Load Balancer)**: Splits traffic between servers
- **EC2**: Web servers running the website
- **Security Groups**: Firewall rules for servers
- **CloudWatch**: Monitor server health

### How to Run Version 1

```bash
cd terraform/v1
terraform init
terraform plan
terraform apply
```

Then visit the ALB URL in EC2 console to see your website.

### Architecture Diagram

```
Internet
   |
   v
Internet Gateway
   |
   v
Load Balancer
   |
   +----+----+
   |         |
   v         v
 Server1   Server2
 (EC2)     (EC2)
```

---

## Version 2: Cost Saving with Spot Instances

### What I Improved

Replaced normal servers with 70% cheap Spot instances + 30% normal instances.

### Why?

- Spot instances are **70% cheaper** than normal ones
- AWS can take them back with 2 minutes notice
- We keep 30% normal servers so website stays up
- Auto Scaling replaces broken servers automatically

### New Concepts

- **Spot Instances**: Cheap unused servers
- **Auto Scaling Group**: Automatically creates/deletes servers
- **Mixed Instances Policy**: Mix of cheap + reliable servers
- **Health Checks**: Makes sure servers are healthy

### How to Run Version 2

```bash
cd terraform/v2
terraform init
terraform plan
terraform apply
```

Watch EC2 console - you'll see servers marked as "Spot" (cheap ones).

### Cost Savings

- Version 1: $0.07 per hour per server = $1.68 per day
- Version 2: 70% Spot + 30% normal = **$0.60 per day** (64% savings!)

---

## Folder Structure

```
aws-highly-available-vpc-architecture/
├── terraform/
│   ├── v1/                    # Version 1 - Basic setup
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   └── v2/                    # Version 2 - Spot + Auto Scaling
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
├── scripts/
│   └── user-data.sh          # Website setup script
└── README.md                  # This file
```
