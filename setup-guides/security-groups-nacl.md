# Security Groups and NACLs Configuration Guide

## Overview
This guide details the security group and network ACL (NACL) configurations for the highly available VPC architecture.

## Security Groups

### ALB Security Group

**Purpose**: Controls traffic to the Application Load Balancer

**Inbound Rules**:
| Protocol | Port Range | Source | Description |
|----------|-----------|--------|-------------|
| HTTP | 80 | 0.0.0.0/0 | Internet HTTP traffic |
| HTTPS | 443 | 0.0.0.0/0 | Internet HTTPS traffic |

**Outbound Rules**:
| Protocol | Port Range | Destination | Description |
|----------|-----------|------------|-------------|
| TCP | 80 | EC2 SG | To application servers |
| TCP | 443 | EC2 SG | To application servers |
| TCP | 3000-9000 | EC2 SG | Custom app ports (adjust as needed) |

**Configuration Steps**:
1. EC2 Dashboard → Security Groups → Create security group
2. Name: `alb-sg`
3. Description: "Security group for Application Load Balancer"
4. VPC: Select your VPC
5. Add inbound rules as above
6. Click "Create"

### EC2 Security Group

**Purpose**: Controls traffic to application EC2 instances

**Inbound Rules**:
| Protocol | Port Range | Source | Description |
|----------|-----------|--------|-------------|
| TCP | 80 | ALB SG | HTTP from ALB |
| TCP | 443 | ALB SG | HTTPS from ALB |
| TCP | 3000-9000 | ALB SG | App ports from ALB |
| TCP | 22 | Bastion SG | SSH from Bastion Host |
| TCP | 22 | 10.0.0.0/16 | SSH from VPC (internal) |

**Outbound Rules**:
| Protocol | Port Range | Destination | Description |
|----------|-----------|------------|-------------|
| All | All | 0.0.0.0/0 | All outbound traffic |

**Configuration Steps**:
1. EC2 Dashboard → Security Groups → Create security group
2. Name: `ec2-sg`
3. Description: "Security group for EC2 application servers"
4. VPC: Select your VPC
5. Add inbound rules as above
6. Click "Create"

### Bastion Host Security Group

**Purpose**: Controls SSH access to the Bastion Host

**Inbound Rules**:
| Protocol | Port Range | Source | Description |
|----------|-----------|--------|-------------|
| TCP | 22 | Your IP/0.0.0.0/0 | SSH from admin |

**Outbound Rules**:
| Protocol | Port Range | Destination | Description |
|----------|-----------|------------|-------------|
| TCP | 22 | EC2 SG | SSH to EC2 instances |

**Configuration Steps**:
1. EC2 Dashboard → Security Groups → Create security group
2. Name: `bastion-sg`
3. Description: "Security group for Bastion Host"
4. VPC: Select your VPC
5. Add inbound rules as above
6. Click "Create"

## Network ACLs (NACLs)

### Public Subnet NACL

**Inbound Rules**:
| Rule # | Type | Protocol | Port Range | Source | Allow/Deny |
|--------|------|----------|-----------|--------|------------|
| 100 | HTTP | TCP | 80 | 0.0.0.0/0 | Allow |
| 110 | HTTPS | TCP | 443 | 0.0.0.0/0 | Allow |
| 120 | SSH | TCP | 22 | 0.0.0.0/0 | Allow |
| 130 | Ephemeral | TCP | 1024-65535 | 0.0.0.0/0 | Allow |
| 140 | Ephemeral | UDP | 1024-65535 | 0.0.0.0/0 | Allow |
| 32767 | All | All | All | 0.0.0.0/0 | Deny |

**Outbound Rules**:
| Rule # | Type | Protocol | Port Range | Destination | Allow/Deny |
|--------|------|----------|-----------|-------------|------------|
| 100 | All | All | All | 0.0.0.0/0 | Allow |

### Private Subnet NACL

**Inbound Rules**:
| Rule # | Type | Protocol | Port Range | Source | Allow/Deny |
|--------|------|----------|-----------|--------|------------|
| 100 | HTTP | TCP | 80 | 10.0.0.0/16 | Allow |
| 110 | HTTPS | TCP | 443 | 10.0.0.0/16 | Allow |
| 120 | SSH | TCP | 22 | 10.0.0.0/16 | Allow |
| 130 | Ephemeral | TCP | 1024-65535 | 0.0.0.0/0 | Allow |
| 140 | Ephemeral | UDP | 1024-65535 | 0.0.0.0/0 | Allow |
| 32767 | All | All | All | 0.0.0.0/0 | Deny |

**Outbound Rules**:
| Rule # | Type | Protocol | Port Range | Destination | Allow/Deny |
|--------|------|----------|-----------|-------------|------------|
| 100 | All | All | All | 0.0.0.0/0 | Allow |

## Least Privilege Principle

The configuration implements least privilege access:
- **Public subnets** allow inbound HTTP/HTTPS for internet-facing traffic
- **EC2 instances** only accept traffic from ALB on application ports
- **Bastion Host** is the only entry point for SSH
- **Private instances** have no direct internet access
- **Ephemeral ports** are allowed for connection responses

## Security Best Practices Applied

1. **Stateful Firewalls**: Security Groups maintain connection state
2. **Deny by Default**: NACLs follow explicit allow pattern
3. **Layered Security**: Both Security Groups and NACLs provide defense
4. **VPC Isolation**: Internal VPC traffic uses private IP ranges
5. **Audit Trail**: Logs can be enabled for NACL and SG analysis

## Troubleshooting

- **No connectivity from internet**: Verify HTTP/HTTPS rules in ALB SG
- **Bastion can't SSH to EC2**: Check EC2 SG allows Bastion SG on port 22
- **EC2 can't reach internet**: Verify NAT Gateway route and NACL ephemeral ports
- **Intermittent connectivity**: Check NACL ephemeral port ranges (1024-65535)
