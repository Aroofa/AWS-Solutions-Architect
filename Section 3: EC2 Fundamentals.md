## EC2
EC2 is one of the most popular of AWS’ offering  
EC2 = Elastic Compute Cloud = Infrastructure as a Service  

It mainly consists in the capability of:
- Renting virtual machines (EC2)
- Storing data on virtual drives (EBS)
- Distributing load across machines (ELB)
- Scaling the services using an auto-scaling group (ASG)

Knowing EC2 is fundamental to understand how the Cloud works

---

## EC2 Sizing & Configuration Options
- Operating System (OS): Linux, Windows or Mac OS  
- How much compute power & cores (CPU)  
- How much random-access memory (RAM)  
- How much storage space:
  - Network-attached (EBS & EFS)
  - Hardware (EC2 Instance Store)
- Network card: speed of the card, Public IP address  
- Firewall rules: security group  
- Bootstrap script (configure at first launch): EC2 User Data  

---

## EC2 User Data
It is possible to bootstrap our instance using an EC2 User Data script  
Bootstrapping means launching commands when a machine starts  
That script is only run once at the instance first start  

EC2 user data is used to automate boot tasks such as:
- Installing updates
- Installing software
- Downloading common files from the internet  
- Anything you can think of  

The EC2 User Data Script runs with the root user  

---

## EC2 Instance Types
You can use different types of EC2 instances that are optimised for different use cases  
AWS has the following naming convention:

`m5.2xlarge`  
- m = instance class  
- 5 = generation (AWS improves them over time)  
- 2xlarge = size within the instance class  

### General Purpose
Great for a diversity of workloads such as web servers or code repositories  
Balance between:
- Compute
- Memory
- Networking  

In the course, we will using the t2.micro which is a general purpose EC2 instance  

### Compute Optimized
Great for compute-intensive tasks that require high performance processors:
- Batch processing workloads  
- Media transcoding  
- High performance web servers  
- High performance computing (HPC)  
- Scientific modeling & machine learning  
- Dedicated gaming servers  

### Memory Optimized
Fast performance for workloads that process large data sets in memory  
Use cases:
- High performance, relational/non-relational databases  
- Distributed web scale cache stores  
- In-memory databases optimized for BI  
- Applications performing real-time processing of big unstructured data  

### Storage Optimized
Great for storage-intensive tasks that require high, sequential read and write access to large data sets on local storage  
Use cases:
- High frequency online transaction processing (OLTP) systems  
- Relational & NoSQL databases  
- Cache for in-memory databases (Redis)  
- Data warehousing applications  
- Distributed file systems  

---

# Security Groups & Classic Ports Overview

Security Groups are the fundamental of network security in AWS  
They control how traffic is allowed into or out of our EC2 Instances  
Security Groups only contain allow rules  
Security Groups rules can reference by IP or by security group  

Security groups act as a “firewall” on EC2 instances  
They regulate:
- Access to Ports  
- Authorised IP ranges - IPv4 and IPv6  
- Control of inbound network (from other to the instances)  
- Control of outbound network (from the instance to other)  

More notes:
- Can be attached to multiple instances  
- Locked down to a region / VPC combination  
- Lives “outside” the EC2  
- All inbound traffic is blocked by default  
- All outbound traffic is authorised by default  
- Maintain one separate security group for SSH  
- If application is not accessible → security group issue  
- If “connection refused” → application error or not launched  

### Classic Ports to Know
22 = SSH  
21 = FTP  
22 = SFTP  
80 = HTTP  
443 = HTTPS  
3389 = RDP  

---

## SSH Summary Table

### SSH works on:
- Mac  
- Linux  
- Windows >= 10  

### Putty works on:
- Windows < 10  
- Windows >= 10  

### EC2 Instance Connect
- Mac  
- Linux  
- Windows < 10  
- Windows >= 10  

### SSH Troubleshooting
Students have the most problems with SSH  
If things don’t work:
- Re-watch the lecture  
- Read the troubleshooting guide  
- Try EC2 Instance Connect  
- If one method works, you’re good  
- If none work, that’s okay — the course won’t use SSH much  

---

# EC2 Instance Purchasing Options

- **On-Demand Instances** - short workload, predictable pricing, pay by second  
- **Reserved (1 & 3 years)**  
  - Reserved Instances - long workloads  
  - Convertible Reserved Instances - long workloads with flexible instances  
- **Saving plans (1 & 3 years)** - commitment to an amount of usage  
- **Spot Instances** - short workloads, cheap, can lose instances  
- **Dedicated Hosts** - entire physical server reserved  
- **Dedicated Instances** - no shared hardware  
- **Capacity reservations** - reserve capacity in a specific AZ  

---

## EC2 On-Demand
Pay for what you use:
- Linux or Windows = billing per second, after the first minute  
- All other operating systems = billing per hour  

Characteristics:
- Highest cost, no upfront payment  
- No long-term commitment  
- Good for short-term, unpredictable workloads  

---

## EC2 Reserved Instances
- Up to 72% discount  
- Reserve a specific instance attribute  
- Reservation: 1 year or 3 years  
- Payment options: No Upfront, Partial, All Upfront  
- Scope: Regional or Zonal  
- Good for steady-state applications  
- Can buy/sell on RI Marketplace  

---

## Convertible Reserved Instances
- Can change EC2 instance type, family, OS, scope, tenancy  
- Up to 66% discount  

---

## EC2 Saving Plans
- Discount based on long-term usage (up to 72%)  
- Commit to usage ($/hour for 1 or 3 years)  
- Beyond commitment billed at On-Demand price  
- Locked to instance family & region  
- Flexible for size, OS, tenancy  

---

## EC2 Spot Instances
- Up to 90% discount  
- Instances can be lost anytime if max price < current spot price  
- Most cost-efficient  
Useful for:
- Batch jobs  
- Data analysis  
- Image processing  
- Distributed workloads  
Not good for:
- Critical jobs  
- Databases  

---

## EC2 Dedicated Hosts
- Entire physical server dedicated  
- Helps with compliance & licensing  
- Pricing: On-demand or Reserved  
- Most expensive option  

---

## EC2 Dedicated Instances
- Hardware dedicated to you  
- May share hardware with other instances in the same account  
- No control over placement  

---

## EC2 Capacity Reservations
- Reserve capacity in specific AZ  
- No commitment  
- Charged On-Demand even if unused  
- Good for short-term workloads that *must* run in a specific AZ  

---

# Which purchasing option is right for me?

- **On-demand:** pay full price whenever you come  
- **Reserved:** plan ahead, stay long-term, get discount  
- **Saving:** commit per hour for a period, use any room type  
- **Spot:** bid for empty rooms, you can get kicked out anytime  
- **Dedicated Hosts:** book an entire building  
- **Capacity Reservations:** pay full price even if you don’t stay  

---

# Spot Instances & Spot Fleet

## Spot instance requests
- Up to 90% discount  
- Define max price  
- If spot price > max, instance terminates or stops (2 min warning)  

### Spot Block
- “Block” instance for 1–6 hours  
- Rare interruptions  
- Good for resilient workloads  
- Not good for critical jobs or databases  

### Terminating Spot Instances
- You can only cancel requests that are open/active/disabled  
- Cancelling request ≠ terminating instance  
- Must cancel request AND terminate instance  

---

## Spot Fleets
Spot Fleets = Spot Instances + optional On-Demand  
Fleet tries to meet target capacity with price constraints  

Launch pools can vary by:
- Instance type  
- OS  
- Availability zone  

Fleet stops when capacity or max cost reached  

### Allocation Strategies
- **lowestPrice** – cheapest pool  
- **Diversified** – spread evenly  
- **capacityOptimized** – optimal capacity  
- **priceCapacityOptimized (recommended)** – best capacity + lowest price  

Spot Fleets allow automatic Spot Instance requests at lowest price  
