# AWS Scalable Web App Cheat Sheet

---

## Lesson 1: WhatIsTheTime.com (Scalable EC2 Architecture)

### 1. Single EC2 Instance
- **Pros:** Simple, cheap; 1 public EC2 + Elastic IP.
- **Cons:** Single point of failure, no scalability, downtime during changes.

### 2. Vertical Scaling
- Increase instance size (t2.micro → m5.large)
- **Requires downtime**
- Limited growth

> **Exam Tip:** Vertical scaling = “bigger box” → downtime.

### 3. Horizontal Scaling (Manual)
- Add EC2 instances manually
- Each instance has its own IP → management headache
- Elastic IPs **do not scale**

### 4. Horizontal Scaling via Route 53 A Records
- Distribute traffic across public IPs
- **Problems:** TTL caching, no health checks
> **Bad design for dynamic EC2 IPs**

### 5. Load Balancer (ALB/ELB)
- Move EC2s to **private subnets**
- Route 53 uses **Alias → LB**
- Manages:
  - Health checks
  - Dynamic IP updates
  - Auto load distribution
  - Zero-downtime deployments

### 6. Auto Scaling Group (ASG)
- Auto launch/terminate EC2s
- Maintains **desired capacity**
- Integrates with LB target groups
- Scales via CPU, CloudWatch, schedules

### 7. Multi-AZ Architecture
- LB & ASG span multiple AZs
- Provides **high availability**, not multi-region redundancy

### 8. Cost Optimization
- Baseline → **Reserved Instances** (predictable)
- Burst → **Spot Instances** (interruptible, cheapest)

**Final Architecture**
    Route 53 (Alias)
            ↓
    Load Balancer (Multi-AZ)
            ↓
    Auto Scaling Group (Multi-AZ)
            ↓
    Private EC2 Instances

---

## Lesson 2: Route 53 & DNS Quick Reference

| Term | Description |
|------|------------|
| **Domain Registrar** | Amazon Route 53, GoDaddy |
| **DNS Records** | A, AAAA, CNAME, NS, MX, TXT, SPF, etc. |
| **Zone File** | Contains DNS records |
| **Name Server** | Resolves DNS queries |
| **TLD** | Top Level Domain (.com, .org) |
| **SLD** | Second Level Domain (example.com) |

### Record Types
- **A / AAAA:** hostname → IPv4 / IPv6
- **CNAME:** hostname → hostname (non-root only)
- **NS:** name servers for the zone
- **Alias:** points to AWS resource, root domain supported, free, auto-updates IPs

### Routing Policies
- Simple, Weighted, Failover, Latency-based, Geolocation, Multi-Value, Geoproximity, IP-based

### Health Checks
- Public: HTTP, HTTPS, TCP endpoints
- Can integrate CloudWatch alarms
- Private: use CloudWatch metric & alarm

> **Exam Tip:** Alias + LB + Health Checks = scalable, HA architecture

---

## Lesson 3: MyWordPress.com (Stateful Web App)

### Database Layer
| Option | Features |
|--------|---------|
| **RDS (MySQL)** | Multi-AZ, standard scaling |
| **Aurora MySQL** | Multi-AZ, read replicas, global DBs, low operational overhead |

> **Tip:** Aurora preferred for high-traffic applications.

### File Storage: Images & Media
#### EBS (Single Instance)
- Works for 1 EC2, 1 AZ
- Multi-AZ → images not shared → breaks WordPress

#### EFS (Shared Storage)
- NFS shared across all EC2 instances
- Creates ENIs in each AZ
- Supports **multi-AZ scaling**
- Shared storage for WordPress uploads

| Feature | EBS | EFS |
|---------|-----|-----|
| Cost | Lower | Higher |
| Multi-AZ | ❌ | ✅ |
| Shared storage | ❌ | ✅ |
| Scalability | Limited | High |

### Architecture Summary
      Client
        ↓
      Route 53
        ↓
      ALB (Multi-AZ)
        ↓
      EC2 Auto Scaling Group (Stateless, Multi-AZ)
        ↓
      Shared File Storage (EFS)
        ↓
      Database (Aurora MySQL or RDS)


**Features**
- Horizontally scalable web tier
- Multi-AZ redundancy
- Shared media storage
- Durable database storage with read replicas
- Centralized session/content handling

---

## Key Exam Tips
- **EBS vs EFS:** EBS for single-instance, EFS for multi-instance scaling
- **Aurora vs RDS:** Aurora preferred for high-traffic apps
- **Cost vs Performance:** EFS = more expensive, critical for multi-AZ
- **Vertical vs Horizontal Scaling:** Horizontal + LB + ASG preferred
- **Multi-AZ ≠ Multi-Region**
- **Route 53 Alias → LB** is the recommended scalable DNS design
