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

## Lesson 4: Instantiating Applications Quickly

### EC2 Launch Speed

1. **Golden AMI**
   - Pre-install OS, apps, dependencies
   - Create an AMI
   - Future EC2 instances launch **ready-to-go**
   - Fastest startup method

2. **User Data / Bootstrapping**
   - Configure instance at startup
   - Install dynamic settings: DB URLs, passwords, etc.
   - Slower than golden AMI for repeated installs

> **Hybrid Approach:** Golden AMI + user data → pre-installed apps + dynamic configuration

### RDS & EBS Snapshots
- **RDS:** Restore from snapshot → database ready with schema and data
- **EBS:** Restore from snapshot → disk pre-formatted with data
- Speeds up deployment and recovery

**Exam Tip:**  
- Use **golden AMI + user data** to speed EC2 launch  
- Use **RDS/EBS snapshots** to speed database and storage initialization

---

## Lesson 5: Elastic Beanstalk (Managed App Deployment)

### What is Elastic Beanstalk?
- Developer-centric deployment service
- Manages underlying infrastructure: EC2, ASG, ELB, RDS, scaling, health monitoring
- Developer provides **only the application code**
- Supports multiple programming languages and container formats

### Components
- **Application:** Collection of environments, versions, and configurations
- **Version:** Iteration of application code
- **Environment:** Collection of resources running a specific version
- **Tiers:** 
  - Web server environment → traditional LB + ASG
  - Worker environment → EC2 instances process **SQS queue** messages

### Deployment Modes
1. **Single Instance**
   - One EC2 instance (with Elastic IP)
   - Suitable for dev/test
2. **High Availability**
   - Multi-AZ LB + ASG
   - Multi-AZ RDS
   - Production-ready deployment

### Key Features
- Supports Dev/Test/Prod environments
- Updates via application versions
- Worker & Web environments can be combined
- Autoscaling based on load (HTTP requests or SQS messages)

---

## Key Takeaways

- **Stateless Web Tier**:
  - ALB + ASG for horizontal scaling
  - Sticky sessions / server-side sessions
  - ElastiCache / DynamoDB for session storage

- **Stateful / Media Apps**:
  - EFS for shared files
  - Aurora MySQL / RDS for user/content storage
  - Read replicas & caching for scaling reads

- **Multi-AZ / HA**:
  - LB, ASG, DB, cache → Multi-AZ
  - Route 53 → highly available DNS

- **Elastic Beanstalk**:
  - Simplifies deployment
  - Auto-manages infrastructure
  - Web + Worker tiers, version control
  - Supports multiple languages and environments

- **Security**:
  - Tight SG rules between tiers
  - Limit public access where possible

- **Cost Optimization**:
  - Reserved + Spot instances
  - Caching reduces DB load
  - EFS higher cost but required for multi-AZ media sharing

- **Rapid Deployment**:
  - Golden AMI + User Data
  - RDS & EBS snapshots for fast restoration

---

**Exam Tips**
- Horizontal scaling + LB + ASG = core HA pattern
- Multi-AZ ≠ Multi-Region
- Stickiness = short-term solution, use server-side session for scale
- ElastiCache = sub-ms session retrieval, reduces DB load
- EFS = shared media for multi-instance apps
- Snapshots & golden AMI = speed up deployments
- Beanstalk = manage infra, deploy code, supports dev/prod easily
