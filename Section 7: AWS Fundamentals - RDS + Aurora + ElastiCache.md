# Lesson 1: Amazon RDS Overview

RDS stands for relational Database Service  
It’s a managed DB service for DB use SQL as a query language  
It allows you to create databases in the cloud that are managed by AWS  
- Postgres  
- MySQL  
- MariaDB  
- Oracle  
- Microsoft SQL Server  
- IBM DB2  
- Aurora (AWS Proprietary database)

### Advantage over using RDS versus deploying DB on EC2
RDS is a managed service:
- Automated provisioning, OS Patching  
- Continuous backups and restore to specific timestamp (Point in Time Restore)!  
- Monitoring dashboards  
- Read replicas for improved read performance  
- Multi AZ setup for DR (Disaster Recovery)  
- Maintenance windows for upgrades  
- Scaling capability (vertical and horizontal)  
- Storage backed by EBS  

But you can’t SSH into your instances.

---

## RDS - Storage Auto Scaling
Helps you increase storage on your RDS DB instance dynamically  
When RDS detects you are running out of free database storage, it scales automatically  
Avoid manually scaling your database storage  

You have to set Maximum Storage Threshold (maximum limit for DB storage)

Automatically modify storage if:
- Free storage is less than 10% of allocated storage  
- Low-storage lasts at least 5 minutes  
- 6 hours have passed since last modification  

Useful for applications with unpredictable workloads  
Supports all RDS database engines  

---

# Lesson 2: RDS Read Replicas vs Multi-AZ

## RDS Read Replicas
RDS Read Replicas for read scalability  
Up to 15 Read Replicas  
Within AZ, Cross AZ or Cross Region  
Replication is ASYNC, so reads are eventually consistent  
Replicas can be promoted to their own DB  
Applications must update the connection string to leverage read replicas  

### RDS Read Replicas - Use Cases
- You have a production database that is taking on normal load  
- You want to run a reporting application to run some analytics  
- You create a Read Replica to run the new workload there  
- The production application is unaffected  
- Read replicas are used for SELECT (=read) only statements (not INSERT/UPDATE/DELETE)

### RDS Read Replicas - Network Cost
In AWS there’s a network cost when data goes from one AZ to another  
For RDS Read Replicas within the same region, you don’t pay that fee  

---

## RDS Multi-AZ (Disaster Recovery)
- SYNC replication  
- One DNS name - automatic app failover to standby  
- Increase availability  
- Failover in case of loss of AZ, network, instance or storage failure  
- No manual intervention in apps  
- Not used for scaling  

**Note:** The Read Replicas can be setup as Multi AZ for Disaster Recovery (DR)

### RDS - From Single-AZ to Multi-AZ
Zero downtime operation  
Just click on “modify” for the database  

Internally:
- A snapshot is taken  
- A new DB is restored from the snapshot in a new AZ  
- Synchronization is established between the two databases  

---

# Lesson 3: RDS Custom For Oracle and Microsoft SQL Server

## RDS Custom
Managed Oracle and Microsoft SQL Server Database with OS and database customization  
RDS: Automates setup, operation, and scaling of database in AWS  
Custom: access to the underlying database and OS so you can:
- Configure settings  
- Install patches  
- Enable native features  
- Access the underlying EC2 Instance using SSH and SSM Session Manager  

De-activate Automation Mode to perform your customization  
Better to take a DB snapshot before

### RDS vs. RDS Custom
- RDS: entire database and the OS managed by AWS  
- RDS Custom: full admin access to underlying OS and database  

---

# Lesson 4: Amazon Aurora

Aurora is a proprietary technology from AWS  
Postgres and MySQL are both supported as Aurora DB (drivers work normally)  
Aurora is AWS cloud optimized and claims:
- 5x performance improvement over MySQL on RDS  
- 3x performance over Postgres on RDS  

Aurora storage grows automatically in 10GB increments up to 128 TB  
Up to 15 replicas, sub-10ms replica lag  
Failover is instantaneous (HA native)  
Costs ~20% more than RDS  

---

## Aurora High Availability and Read Scaling

6 copies of your data across 3 AZ:
- 4/6 needed for writes  
- 2/6 needed for reads  

Self healing with peer-to-peer replication  
Storage striped across 100s of volumes  

One Aurora Instance takes writes (master)  
Automated failover < 30 seconds  
Master + up to 15 read replicas  

Supports Cross Region Replication  

---

## Features of Aurora
- Automatic fail-over  
- Backup and Recovery  
- Isolation and Security  
- Industry compliance  
- Push-button scaling  
- Automated patching with Zero Downtime  
- Advanced Monitoring  
- Routine Maintenance  
- Backtrack restore without backups  

---

# Lesson 5: Aurora – Advanced Concepts

## Aurora Replicas - Auto Scaling
Automatically scale read replicas.

## Aurora - Custom Endpoints
Define a subset of Aurora instances as a custom endpoint  
Example: analytical queries go to specific replicas  

Reader endpoint usually not used after custom endpoints

---

## Aurora Serverless
- Automated instantiation and auto-scaling  
- Good for infrequent, intermittent or unpredictable workloads  
- No capacity planning  
- Pay per second  

---

## Global Aurora

### Aurora Cross Region Read Replicas
- Used for DR  
- Simple setup  

### Aurora Global Database (recommended)
- 1 primary region (read/write)  
- Up to 10 secondary regions (read-only), < 1 second lag  
- Up to 16 read replicas per region  
- Low latency  
- RTO < 1 minute  
- Cross-region replication < 1 second  

---

## Aurora Machine Learning
Integration between Aurora and AWS ML services  
Supported:
- Amazon SageMaker (any ML model)  
- Amazon Comprehend (sentiment analysis)  

No ML experience needed  

Use cases:
- fraud detection  
- ads targeting  
- sentiment analysis  
- product recommendations  

---

## Babelfish for Aurora PostgreSQL
Allows Aurora PostgreSQL to understand SQL Server (T-SQL) commands  
MS SQL Server apps can work with Aurora PostgreSQL  
Minimal code changes  
Useful for migrations using SCT + DMS  

---

# Lesson 6: RDS & Aurora – Backup and Monitoring

## RDS Backups
Automated backups:
- Daily full backup  
- Transaction logs every 5 minutes  
→ restore to any point-in-time  

Retention: 1–35 days (0 disables)

### Manual DB Snapshots
Manually triggered  
Retained indefinitely  

**Trick:** Stopped RDS DB still costs storage → snapshot & restore instead  

---

## Aurora Backups
Automated backups  
1–35 days (cannot disable)  
PITR supported  

### Manual Snapshots
Same as RDS  

---

## Restore Options
Restoring creates a new DB instance.

### Restoring MySQL RDS from S3
- Backup on-prem DB  
- Upload to S3  
- Restore onto RDS  

### Restoring MySQL Aurora from S3
- Use Percona XtraBackup  
- Store on S3  
- Restore to Aurora cluster  

---

## Aurora Database Cloning
Create a new Aurora cluster from an existing one  
Faster than snapshot/restore  
Uses copy-on-write  
Efficient and cost effective  
Great for staging vs production  

---

# Lesson 7: RDS Security

- At-rest encryption with KMS  
- Must be set at launch; read replicas follow master’s encryption state  
- To encrypt unencrypted DB: snapshot → restore as encrypted  
- In-flight encryption using TLS  
- IAM Authentication  
- Security groups control access  
- No SSH except RDS Custom  
- Audit logs → CloudWatch  

---

# Lesson 8: RDS Proxy

Fully managed DB proxy  
Pools and shares DB connections  
Reduces stress on database resources  
Serverless, autoscaling, multi-AZ  
Cuts failover time by up to 66%  
Supports RDS (MySQL, PostgreSQL, MariaDB, SQL Server) & Aurora  
No code changes for most apps  
IAM authentication, Secrets Manager  
Never publicly accessible (VPC only)  

---

# Lesson 9: ElastiCache Overview

RDS → managed relational DB  
ElastiCache → managed Redis or Memcached  

Caches = in-memory DBs  
Low latency, high performance  
Reduce load on RDS  
Help make applications stateless  

AWS handles:
- OS patching  
- Optimization  
- Setup  
- Monitoring  
- Failover  
- Backups  

Using ElastiCache usually requires application code changes.

---

## ElastiCache Architecture – DB Cache
Application checks cache → if miss → go to RDS → store in cache  
Requires cache invalidation strategy  

---

## ElastiCache Architecture – Session Store
Application stores user session in cache  
Another instance retrieves → user stays logged in  

---

## ElastiCache – Redis vs Memcached

### Redis
- Multi AZ with Auto-Failover  
- Read Replicas  
- Durability (AOF)  
- Backup & Restore  
- Supports sets and sorted sets  

### Memcached
- Multi-node partitioning (sharding)  
- No replication  
- Non persistent  
- Backup/restore (Serverless)  
- Multi-threaded  

---

# Lesson 10: ElastiCache for Solution Architects

## Cache Security
Redis:
- IAM Authentication  
- Redis AUTH token  
- SSL in-flight encryption  

Memcached:
- SASL authentication  

---

## Cache Patterns
- **Lazy Loading:** read data cached, can become stale  
- **Write Through:** update cache when DB updates (no stale data)  
- **Session Store:** short-term user data  

Quote:  
**“There are only two hard things in Computer Science: cache invalidation and naming things.”**

---

## Redis Use Case
Gaming leaderboards  
Sorted sets maintain order + uniqueness  
Real-time ranking  
