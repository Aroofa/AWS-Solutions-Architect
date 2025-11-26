## Lesson 1: WhatIsTheTime.com (Stateless Web App)

### Architecture Patterns

1. **Single EC2 Instance**
   - Pros: Simple, cheap; 1 public EC2 + Elastic IP.
   - Cons: Single point of failure, no scalability, downtime during changes.

2. **Vertical Scaling**
   - Increase instance size (t2.micro → m5.large)
   - **Requires downtime**
   - Limited growth
   > **Exam Tip:** Vertical scaling = “bigger box” → downtime.

3. **Horizontal Scaling (Manual)**
   - Add EC2 instances manually
   - Each instance has its own IP → management headache
   - Elastic IPs **do not scale**

4. **Route 53 A Records**
   - Distribute traffic across public IPs
   - Problems: TTL caching, no health checks
   > **Bad design for dynamic EC2 IPs**

5. **Load Balancer (ALB/ELB)**
   - Move EC2s to private subnets
   - Route 53 uses **Alias → LB**
   - LB manages:
     - Health checks
     - Dynamic IP updates
     - Auto load distribution
     - Zero-downtime deployments

6. **Auto Scaling Group (ASG)**
   - Automatically launch/terminate EC2s
   - Maintains **desired capacity**
   - Scales via CPU, CloudWatch, schedules
   - Integrates with LB target groups

7. **Multi-AZ Architecture**
   - LB & ASG span multiple AZs
   - Provides **high availability**, not multi-region redundancy

8. **Cost Optimization**
   - Baseline → **Reserved Instances** (predictable)
   - Burst → **Spot Instances** (interruptible, cheapest)

**Architecture Diagram**
        Route 53 (Alias)
        ↓
        Load Balancer (Multi-AZ)
        ↓
        Auto Scaling Group (Multi-AZ)
        ↓
        Private EC2 Instances
        
---

## Lesson 2: MyClothes.com (Stateful Web App / Session Management)

### Problem
- Users have shopping carts.
- Horizontal scaling → requests can go to different EC2 instances → **shopping cart lost**.
- Goal: Keep web tier **stateless** while preserving session state.

### Solutions

1. **Sticky Sessions / Session Affinity**
   - ELB remembers which instance the user talks to.
   - Pros: Simple, preserves cart per instance.
   - Cons: If EC2 instance terminates → cart lost.

2. **Client-side Cookies**
   - Store shopping cart data in user cookies.
   - Pros: Web tier is stateless.
   - Cons: Max size ~4KB, security risks (data can be altered), HTTP requests heavier.

3. **Server-side Session (Recommended)**
   - Send **Session ID** to client.
   - Session data stored in **ElastiCache (Redis/Memcached)**.
   - EC2 instances fetch session data using Session ID.
   - Pros: Stateless EC2, sub-millisecond access, secure.
   - Alternative: **DynamoDB** for session storage.

### User Data Persistence
- Store user data (address, name) in **RDS / Aurora**
- Long-term storage, accessible by all instances

### Scaling Reads
1. **RDS Read Replicas**
   - Scale reads up to 15 replicas.
2. **Lazy Loading / Cache**
   - Use **ElastiCache** to cache frequently read data.
   - Reduces RDS load, improves performance.
   - Application-side cache maintenance required.

### High Availability / Multi-AZ
- Multi-AZ ELB & ASG → survive AZ failure
- RDS → Multi-AZ with standby replica
- ElastiCache (Redis) → Multi-AZ
- Route 53 → highly available DNS

### Security Groups
- ALB: HTTP/HTTPS open to world
- EC2: Allow only traffic from ALB
- ElastiCache: Allow only traffic from EC2
- RDS: Allow only traffic from EC2

**Architecture Diagram**
        Client
        ↓
        Route 53
        ↓
        ALB (Multi-AZ)
        ↓
        EC2 Auto Scaling Group (Stateless, Multi-AZ)
        ↓
        ElastiCache (Session / Cache)
        ↓
        Database (RDS / Aurora)


---

## Lesson 3: MyWordPress.com (Stateful Media Web App)

### Database Layer
| Option | Features |
|--------|---------|
| RDS (MySQL) | Multi-AZ, standard scaling |
| Aurora MySQL | Multi-AZ, read replicas, global DB, lower operational overhead |

> **Tip:** Aurora preferred for high-traffic websites.

### File Storage
**Problem with EBS**
- Single instance: fine
- Multiple instances / AZs → images not shared → broken WordPress

**Solution: EFS**
- NFS shared across EC2s
- Creates ENIs in each AZ
- Multi-AZ scalable
- Shared storage for media uploads

| Feature | EBS | EFS |
|---------|-----|-----|
| Cost | Lower | Higher |
| Multi-AZ | ❌ | ✅ |
| Shared storage | ❌ | ✅ |
| Scalability | Limited | High |

**Architecture Diagram**
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

  
---

## Key Takeaways

- **Stateless Web Tier**:
  - Use ALB + ASG for horizontal scaling.
  - Sticky sessions or server-side sessions for user state.
  - ElastiCache or DynamoDB for session storage.

- **Stateful / Media Apps**:
  - Use EFS for shared files.
  - Aurora MySQL / RDS for user and content storage.
  - Read replicas & caching for scaling reads.

- **Multi-AZ / HA**:
  - Always deploy LB, ASG, DB, and cache in Multi-AZ.
  - Route 53 + Alias → LB → EC2 ensures resilient architecture.

- **Security**:
  - Tight SG rules between tiers.
  - Limit access from the internet where possible.

- **Cost Optimization**:
  - Reserved + Spot instances
  - Caching reduces DB load
  - EFS higher cost but required for multi-AZ media sharing

---

**Exam Tips**
- Horizontal scaling + LB + ASG = core high-availability pattern
- Multi-AZ ≠ Multi-Region
- Stickiness = short-term solution, use server-side session for scale
- ElastiCache = sub-ms session retrieval, reduces DB load
- EFS = shared media for multi-instance apps

