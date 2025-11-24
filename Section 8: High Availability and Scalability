# Lesson 1: High Availability and Scalability  

## Scalability
Scalability means that an application/system can handle greater loads by adapting.  
There are two kinds of scalability:
- Vertical Scalability  
- Horizontal Scalability (= elasticity)

Scalability is linked but different to High Availability.

### Vertical Scalability
Vertically scalability means increasing the size of the instance.  
For example, your application runs on a t2.micro.  
Scaling that application vertically means running it on a t2.large.  
Vertical scalability is very common for non distributed systems, such as a database.  
RDS, ElasticCache are services that can scale vertically.  
There’s usually a limit to how much you can vertically scale (hardware limit).  

### Horizontal Scalability
Horizontal Scalability means increasing the number of instances/systems for your application.  
Horizontal scaling implies distributed systems.  
This is very common for web applications/modern applications.  
It’s easy to horizontally scale thanks to cloud offerings such as Amazon EC2.

## High Availability
High availability usually goes hand in hand with horizontal scaling.  
High availability means running your application/system in at least 2 data centers (== Availability Zones).  
The goal of high availability is to survive a data center loss.  
The high availability can be passive (for RDS Multi AZ for example).  

## High Availability & Scalability for EC2
- Vertical scaling increases instance size (= scale up/down).  
  - From t2.nano – 0.5G of RAM, 1 vCPUs  
  - To u-12tb1.metal – 12.3 TB of RAM, 448 vCPUs  
- Horizontal Scaling: Increase number of instances (= scale out/in)  
  - Auto Scaling Group  
  - Load Balancer  
- High Availability: Run instances for the same application across multi AZ  
  - Auto Scaling Group multi AZ  
  - Load Balancer multi AZ  


---

# Lesson 2: Elastic Load Balancing (ELB) Overview

## Scalability & Availability  
Scalability means that an application/system can handle greater loads by adapting.  
Two kinds of scalability:
- Vertical Scalability  
- Horizontal Scalability (= elasticity)

Scalability is linked but different to High Availability.

### Vertical Scalability
Vertically scalability means increasing the size of the instance.  
For example, your application runs on a t2.micro.  
Scaling that application vertically means running it on a t2.large.  
Vertical scalability is common for non distributed systems (e.g., a database).  
RDS, ElastiCache can scale vertically.  
There is usually a hardware limit.

### Horizontal Scalability
Horizontal Scalability means increasing the number of instances/systems for your application.  
Horizontal scaling implies distributed systems.  
Very common for web/modern applications.  
Easy with cloud offerings such as Amazon EC2.

## High Availability
High availability usually goes with horizontal scaling.  
Means running the application in at least 2 AZs.  
Goal: survive a data center loss.  
Can be passive or active depending on the service.

## High Availability & Scalability for EC2
Vertical Scaling:
- From t2.nano - 0.5G RAM, 1 vCPU  
- To 12tb1.metal - 12.3 TB RAM, 448 vCPUs  

Horizontal Scaling:
- Auto Scaling Group  
- Load Balancer  

High Availability (multi AZ):
- Auto Scaling Group multi AZ  
- Load Balancer multi AZ  

---

## What is Load Balancing?
Load balancers are servers that forward traffic to multiple downstream servers (EC2 instances).

### Why use a load balancer?
- Spread load across downstream instances  
- Expose single point of access (DNS)  
- Handle downstream instance failures  
- Health checks  
- Provide SSL termination (HTTPS)  
- Enforce stickiness with cookies  
- High availability across zones  
- Separate public/private traffic  

## Why use an Elastic Load Balancer?
- Managed service (AWS guarantees uptime)  
- Automatic maintenance/upgrades  
- Fewer configuration knobs  
- Cheaper to self-host but requires more effort  
- Integrates with EC2, ASG, ECS, ACM, CloudWatch, Route 53, WAF, Global Accelerator  

---

## Health Checks
Health Checks are crucial for Load Balancers.  
- Check availability of instances  
- Health check uses a port + route (e.g., `/health`)  
- Response != 200 → unhealthy  

---

## Types of Load Balancers (AWS)
AWS has 4 kinds:
- **Classic Load Balancer (CLB)** – 2009  
  - HTTP, HTTPS, TCP, SSL  
- **Application Load Balancer (ALB)** – 2016  
  - HTTP, HTTPS, WebSocket  
- **Network Load Balancer (NLB)** – layer 4  
  - TCP, UDP, ultra-low latency  
- **Gateway Load Balancer (GWLB)** – layer 3  
  - For network appliances  

Recommended: newer generation load balancers.  
Load balancers can be internal or external.

---

# Lesson 3: Application Load Balancer (v2)

ALB is Layer 7 (HTTP).  
Supports:
- Load balancing multiple HTTP apps across machines  
- Load balancing multiple apps on same machine (containers)  
- HTTP/2 & WebSocket  
- Redirects (HTTP → HTTPS)  

### Routing
- Path-based  
- Hostname-based  
- Query string or header-based  

Great for microservices and container apps (ECS).  
Supports port mapping.

### ALB Target Groups
- EC2 instances  
- ECS tasks  
- Lambda functions  
- Private IPs  

### Notes  
- Fixed hostname: `xxx.region.elb.amazonaws.com`  
- Real client IP available via `X-Forwarded-For`  

---

# Lesson 4: Network Load Balancer

Layer 4.  
Supports:
- TCP & UDP traffic  
- Millions of requests per second  
- Ultra-low latency  
- Static IP per AZ + optional Elastic IP  

Used for extreme performance.

### NLB Target Groups
- EC2 Instances  
- Private IPs  

Health checks: TCP, HTTP, HTTPS.

---

# Lesson 5: Gateway Load Balancer

Used to deploy & scale 3rd-party network appliances.  
Examples: firewalls, IDS/IPS, packet inspection.

Operates at Layer 3.  
Combines:
- Transparent Network Gateway  
- Load Balancer  

Uses GENEVE protocol (port 6081).

### Target Groups
- EC2 Instances  
- Private IPs  

---

# Lesson 6: Sticky Sessions

Sticky Session = Session Affinity.  
Ensures same client always hits same instance.

Works with CLB, ALB, NLB.  
Cookie has expiration.

Use case: keep session data consistent.

May cause load imbalance.

### Cookie types
- **Application-based (custom)**  
- **Application cookie (`AWSALBAPP`)**  
- **Duration-based (`AWSALB`, `AWSSELB`)**  

---

# Lesson 7: Cross Zone Load Balancing

With CZLB:
- Each LB node distributes traffic evenly across *all* instances in *all* AZs.

Without CZLB:
- Traffic stays within the node’s own AZ.

### Defaults & Charges
- ALB: enabled by default, no charge  
- NLB/GWLB: disabled by default, inter-AZ data charged  
- CLB: disabled by default, no charge  

---

# Lesson 8: SSL Certificates (ELB)

SSL/TLS encrypt in-flight traffic.  
Public certs issued by CAs like Digicert, GoDaddy, LetsEncrypt.

Load balancers use **X.509 certificates**.  
Use **ACM** to manage or upload your own.

### SNI (Server Name Indication)
Allows multiple certificates on one server.  
Works with ALB, NLB, CloudFront.  
Not with CLB.

### Support per Load Balancer
- **CLB**: one cert only  
- **ALB**: multiple certs, multiple listeners (supports SNI)  
- **NLB**: multiple certs  

---

# Lesson 9: Connection Draining

Names:
- CLB → Connection Draining  
- ALB/NLB → Deregistration Delay  

Allows in-flight requests to finish while instance deregisters.  
Range: 1–3600s (default 300).  

---

# Lesson 10: Auto Scaling Groups (ASG)

ASG goals:
- Scale out during high load  
- Scale in during low load  
- Maintain min/max instance count  
- Register new instances to LB  
- Replace unhealthy instances  

ASG itself is free.

### ASG Attributes
- Launch Template  
- AMI + Instance Type  
- User Data  
- EBS Volumes  
- SGs  
- Key Pair  
- IAM Role  
- Network/Subnets  
- Load Balancer  
- Min/Max/Desired capacity  

### CloudWatch Alarms
Scale based on metrics (CPU, network, custom metrics).

---

# Lesson 11: Scaling Policies

## Dynamic Scaling
### Target Tracking  
Example: maintain 40% average CPU.

### Simple / Step Scaling  
Example:  
- CPU > 70% → add 2  
- CPU < 30% → remove 1  

## Scheduled Scaling  
Example: Fridays at 5pm increase min capacity to 10.

## Predictive Scaling  
Forecast load and scale in advance.

### Good Metrics  
- CPUUtilization  
- RequestCountPerTarget  
- Network In/Out  
- Custom CloudWatch metrics  

## Cooldowns  
Default: 300s.  
Avoid scaling again until metrics stabilize.

Advice: use a ready-to-use AMI to reduce time to serve traffic.  
