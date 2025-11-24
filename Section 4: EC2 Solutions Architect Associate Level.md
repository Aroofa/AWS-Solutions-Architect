# EC2 Solutions Architect Associate Level

## Private vs Public vs Elastic IP

Networking has two sorts of IPs: IPV4 and IPV6  
IPV4: 1.160.10.240  
IPV6: 1900:4545:3:200:f8ff:fe21:67cf  

In this course we will only be using IPV4  
IPV4 is still the most common format used online  
IPV6 is newer and solves problems for the Internet of Things (IoT)  
IPV4 allows for 3.7 billion different addresses in the public space  
IPV4: \[0-255\].\[0-255\].\[0-255\].\[0-255\]

### Public IP
- Public IP means the machine can be identified on the internet (WWW)  
- Must be unique across the whole web (not two machine can have the same public IP)  
- Can be geo-located easily  

### Private IP
- Private IP means the machine can only be identified on a private network only  
- The IP must be unique across the private network  
- BUT two different private networks (two companies) can have the same IPs  
- Machines connect to WWW using an internet gateway (a proxy)  
- Only a specified range of IPs can be used as private IP  

### Elastic IPs
- When you stop and then start an EC2 instance, it can change its public IP  
- If you need to have a fixed public IP for your instance, you need an elastic IP  
- An elastic IP is a public IPV4 IP you own as long as you don’t delete it  
- You can attach it to one instance at a time  
- You can mask failures by remapping the address to another instance  
- You can only have 5 elastic IP in your account (you can ask AWS to increase that)  
- Avoid elastic IPs (often reflect poor architectural decisions)  
- Instead use a random public IP and register a DNS name to it  

### By default, EC2 comes with:
- A private IP for the internal AWS network  
- A public IP for the WWW  

### SSH Access
- You can’t use a private IP unless you’re in the same network  
- You must use the public IP  
- Public IP can change after stop/start  

---

## Placement Groups

Sometimes you want control over the EC2 Instance placement strategy.  
Three types:

### Cluster
- Clusters instances into a low-latency group in a single AZ  
- **Pros:** Great network (10 Gbps bandwidth)  
- **Cons:** If the AZ fails, all instances fail  
- **Use case:** Big Data jobs, low-latency applications  

### Spread
- Spreads instances across underlying hardware (max 7 per AZ per group)  
- **Pros:** Can span AZs, reduces failure risk  
- **Cons:** 7 instances per AZ limit  
- **Use case:** Critical applications, high availability  

### Partition
- Up to 7 partitions per AZ  
- Can span multiple AZs  
- Up to hundreds of EC2 instances  
- Instances in one partition don't share racks with others  
- **Use case:** HDFS, HBase, Cassandra, Kafka  

---

## Elastic Network Interfaces (ENI)

Logical component in a VPC representing a virtual network card.

ENI can have:
- Primary private IPv4 + secondary IPv4s  
- One Elastic IP per private IPv4  
- One public IPv4  
- One or more security groups  
- MAC address  

You can create ENIs separately and attach/move them on the fly.  
Bound to a specific AZ.

---

## EC2 Hibernate

We can stop or terminate instances:  
- **Stop:** EBS data kept  
- **Terminate:** Root EBS deleted (if configured)

### When starting:
- First start: OS boots + EC2 user data runs  
- Later starts: OS boots + apps warm up  

### Hibernate:
- RAM state is preserved  
- Faster boot (OS not restarted)  
- RAM is saved to encrypted root EBS volume  

### Use Cases:
- Long-running processing  
- Saving RAM state  
- Services that take time to initialize  

### Requirements:
- Supported families: C3, C4, C5, I3, M3, M4, R3, R4, T2, T3…  
- RAM < 100 GB  
- Not supported for bare metal  
- AMIs: Amazon Linux 2, Ubuntu, RHEL, CentOS, Windows  
- Root volume must be encrypted EBS  
- Works with On-Demand, Reserved, Spot  
- Cannot hibernate more than 60 days  

