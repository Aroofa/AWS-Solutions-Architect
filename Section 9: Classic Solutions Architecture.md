## Lesson 1: WhatIsTheTime.com 
### 1. Single EC2 Instance (Start Small)
- **Simple, cheap**, uses 1 public EC2 + Elastic IP.
- **Cons:** Single point of failure, no scalability, downtime during changes.

### 2. Vertical Scaling (Scale Up)
- Increase instance size (t2.micro → m5.large).
- **Always requires downtime.**  
- Works only for limited growth.
**Exam Tip:** Vertical scaling = “bigger box” → downtime.

---

### 3. Manual Horizontal Scaling (Scale Out)
- Add more EC2 instances manually.
- Each has its own public IP / Elastic IP.
- **Problems:** IP limits, user confusion, difficult management.
**Exam Tip:** Elastic IPs do NOT scale.

---

### 4. Horizontal Scaling via Route 53 A Records
- Route 53 distributes traffic across instance public IPs.
- **Problems:** TTL caching → stale IPs; no health checks.
**Exam Tip:** A-records + dynamic EC2 IPs = bad design.

---

### 5. Introduce a Load Balancer (ALB/ELB)
- Move EC2 instances to **private subnets**.
- Route 53 uses **Alias** → Load Balancer (LB).
- LB manages:
  - Health checks  
  - Dynamic IP updates  
  - Automatic load distribution  
  - Zero-downtime deployments  
**Exam Tip:** Use **Alias → Load Balancer** for scalable architectures.

---

### 6. Auto Scaling Group (ASG)
- Automatically launches/terminates EC2 instances.
- Maintains **desired capacity** and scales via:
  - CPU
  - CloudWatch metrics
  - Schedules
- Integrates with Load Balancer target groups.
**Exam Tip:** ALB + ASG = core high-availability pattern.

---

### 7. Multi-AZ Architecture
- LB and ASG span multiple availability zones.
- If one AZ fails, traffic flows to the others.
- Provides **high availability**, not multi-region redundancy.
**Exam Tip:** Multi-AZ ≠ Multi-Region.

---

### 8. Cost Optimization: Reserved + Spot Instances
- Baseline capacity (e.g., 2 instances) → **Reserved Instances / Savings Plans** (up to 72% savings).
- Extra burst capacity → **Spot instances** (cheapest, interruptible).
**Exam Tip:**  
- Reserved = predictable workloads.  
- Spot = flexible, interruptible workloads.

---

## Final Architecture Overview

**Route 53 (Alias)**  → **Load Balancer (Multi-AZ)**  → **Auto Scaling Group (Multi-AZ)**  → **Private EC2 Instances**

### Features
- Highly available  
- Fault tolerant  
- Scalable horizontally  
- Auto-healing  
- Cost optimized  
- Zero-downtime scaling  

---
