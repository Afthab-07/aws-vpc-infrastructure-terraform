# AWS Highly Available VPC Architecture - Detailed Description

## Overview
This architecture demonstrates a production-grade, highly available network infrastructure on AWS spanning two Availability Zones (AZs). It implements industry best practices for security, scalability, and fault tolerance.

## Architecture Components

### 1. Virtual Private Cloud (VPC)
- **CIDR Block**: Custom IP range for the entire network
- **Multi-AZ Deployment**: Resources distributed across 2 AZs for high availability
- **Isolated Network**: Private network independent from the public internet

### 2. Subnets

#### Public Subnets (AZ-1 and AZ-2)
- **Purpose**: Host Internet-facing resources (ALB, Bastion Host)
- **Route Table**: Routes traffic to Internet Gateway
- **Auto-assign Public IP**: Enabled for instances to receive public IPs
- **CIDR Allocation**: /24 subnets in each AZ

#### Private Subnets (AZ-1 and AZ-2)
- **Purpose**: Host application servers and databases
- **Route Table**: Routes traffic through NAT Gateway for outbound internet access
- **No Direct Internet Access**: Enhanced security posture
- **CIDR Allocation**: /24 subnets in each AZ

### 3. Internet Gateway (IGW)
- Enables communication between VPC and the internet
- Attached to public subnets
- Provides high availability through AWS-managed infrastructure

### 4. NAT Gateway
- Deployed in public subnets
- Allows private instances to initiate outbound connections
- Prevents inbound traffic to private instances
- Uses Elastic IP for consistent public IP address

### 5. Application Load Balancer (ALB)
- **Placement**: Public subnets across both AZs
- **Function**: Distributes incoming traffic across EC2 instances
- **Health Checks**: Monitors instance health and routes traffic to healthy instances
- **Target Groups**: Routes requests to backend EC2 instances
- **Sticky Sessions**: Optional session persistence

### 6. Auto Scaling Group (ASG)
- **Min/Desired/Max Capacity**: Configurable for cost optimization
- **Launch Template**: Defines instance configuration (AMI, instance type, security groups)
- **Scaling Policies**: CPU-based or custom metric scaling
- **Health Checks**: Replaces unhealthy instances automatically
- **Multi-AZ**: Instances distributed across AZs

### 7. EC2 Instances
- **Application Servers**: Run in private subnets
- **Bastion Host**: SSH gateway in public subnet for secure access to private instances
- **Security**: Communicate with ALB and each other through Security Groups

### 8. Security Groups

#### ALB Security Group
- **Inbound**: Allow HTTP (80) and HTTPS (443) from 0.0.0.0/0
- **Outbound**: Allow traffic to EC2 security group on application port

#### EC2 Security Group
- **Inbound**: Allow traffic from ALB security group
- **Inbound**: Allow SSH from Bastion Host security group
- **Outbound**: Allow necessary protocols for application functionality

#### Bastion Host Security Group
- **Inbound**: Allow SSH (22) from specific IPs (recommended) or 0.0.0.0/0
- **Outbound**: Allow SSH to EC2 security group

### 9. Network ACLs (NACLs)
- Stateless firewall at subnet level
- Inbound rules: Allow required protocols from authorized CIDR blocks
- Outbound rules: Allow responses and necessary outbound traffic
- Ephemeral port range: 1024-65535

### 10. CloudWatch Monitoring
- **CPU Utilization**: Tracks instance CPU usage
- **Network Traffic**: Monitors inbound/outbound data
- **Health Checks**: Monitors ALB and instance health
- **Custom Metrics**: Application-specific monitoring
- **Alarms**: Triggers for scaling and notifications
- **Dashboards**: Centralized view of architecture health

## Traffic Flow

1. **Inbound Traffic**
   - Internet traffic reaches ALB on port 80/443
   - ALB performs health checks on target EC2 instances
   - Healthy traffic is routed to available instances
   - Load distributed across AZs for redundancy

2. **Instance Communication**
   - Private EC2 instances receive traffic from ALB
   - Instances communicate with each other via private IPs
   - No direct internet exposure

3. **Outbound Traffic**
   - Applications send requests through NAT Gateway
   - NAT Gateway translates private IPs to Elastic IP
   - Responses routed back to originating instance

4. **Management Access**
   - SSH access through Bastion Host only
   - Bastion in public subnet bridges to private instances
   - Audit trail of all administrative access

## High Availability & Fault Tolerance

- **Multi-AZ Deployment**: Survives single AZ failure
- **ALB**: Automatically distributes traffic across healthy instances
- **Auto Scaling**: Replaces failed instances automatically
- **Health Checks**: Monitors and removes unhealthy instances
- **Elastic IPs**: Consistent public addresses for NAT and Bastion

## Security Best Practices Implemented

1. **Least Privilege Access**: Security Groups restrict to minimum required ports
2. **Network Isolation**: Public/Private subnet separation
3. **No Direct Access**: Private instances inaccessible directly from internet
4. **Bastion Host**: Single entry point for administrative access
5. **Stateful Firewalls**: Security Groups are stateful
6. **Monitoring**: CloudWatch logs and alarms track all activity
