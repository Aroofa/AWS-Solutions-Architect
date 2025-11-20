# Section 7: EC2 Instance Storage

---

## Lesson 1: EBS Overview

### What’s an EBS Volume?
- An EBS (Elastic Block Store) Volume is a network drive you can attach to your instances while they run  
- It allows your instances to persist data, even after their termination  
- They can only be mounted to one instance at a time (at the CCP level)  
- They are bound to a specific availability zone  
- Analogy: Think of them as a “network USB stick”  

### EBS Volume
- It’s network drive (i.e. not a physical drive)  
- It uses the network to communicate the instance, which means there might be a bit of latency  
- It can be detached from an EC2 instance and attached to another one quickly  
- It’s locked to an Availability Zone (AZ)  
- An EBS Volume in us-east-1a cannot be attached to us-east-1b  
- To move a volume across, you first need to snapshot it  
- Have a provisioned capacity (size in GBs, and IOPS)  
- You get billed for all the provisioned capacity  
- You can increase the capacity of the drive over time  

### EBS - Delete on Termination attribute
- Controls the EBS behavior when an EC2 instance terminates  
- By default, the root EBS volume is deleted (attribute enabled)  
- By default, any other attached EBS volume is not deleted (attribute disabled)  
- This can be controlled by the AWS console / AWS CLI  
- Use Case: preserve root volume when instance is terminated  

---

## Lesson 2: EBS Snapshot

- Make a backup (snapshot) of your EBS volume at a point in time  
- Not necessary to detach volume to do snapshot, but recommend  
- Can copy snapshot across AZ or Region  

### Features:
**EBS Snapshot Archive**  
- Move a snapshot to an “archive tier” that is 75% cheaper  
- Takes within 24 to 72 hours for restoring the archive  

**Recycle Bin for EBS Snapshots**  
- Setup rules to retain deleted snapshots so you can recover them after an accidental deletion  
- Specify retention (from 1 day to 1 year)  

**Fast Snapshot Restore (FSR)**  
- Force full initialization of snapshot to have no latency on the first use ($$$)  

---

## Lesson 3: AMI Overview

- AMI = Amazon Machine Image  
- AMI are a customization of an EC2 instance  
- You add your own software, configuration, operating system, monitoring…  
- Faster boot / configuration time because all your software is pre-packaged  
- AMI are built for a specific region (and can be copied across regions)  

### You can launch EC2 instances from:
- A Public AMI: AWS provided  
- Your own AMI: you make and maintain them yourself  
- An AWS Marketplace AMI: an AMI someone else made (and potentially sells)  

### AMI Process (from an EC2 instance)
- Start an EC2 instance and customize it  
- Stop the instance (for data integrity)  
- Build an AMI - this will also create EBS snapshots  
- Launch instances from other AMIs  

---

## Lesson 4: EC2 Instance Store

- EBS volumes are network drives with good but “limited” performance  
- If you need high-performance hardware disk, use EC2 Instance Store  
- Better I/O performance  
- EC2 Instance Store lose their storage if they’re stopped (ephemeral)  
- Good for buffer / cache / scratch data / temporary content  
- Risk of data loss if hardware fails  
- Backups and Replication are your responsibility  

---

## Lesson 5: EBS Volume Types

EBS volumes come in 6 types:

- **gp2 / gp3 (SSD):** General Purpose SSD  
- **io1 / io2 Block Express (SSD):** Highest performance  
- **st1 (HDD):** Throughput-optimized  
- **sc1 (HDD):** Cold storage  

EBS Volumes are characterized in **Size | Throughput | IOPS**

### General Purpose SSD
- Cost effective storage. Low-latency  
- System boot volumes, Virtual desktops, Development and test environments  
- 1 GiB - 16 TiB  

**gp3:**  
- Baseline of 3000 IOPS and 125 MiB/s  
- Can increase to 16,000 IOPS and 1000 MiB/s  

**gp2:**  
- Can burst to 3000 IOPS  
- Size and IOPS linked (max 6000)  
- 3 IOPS per GB  

### Provisioned IOPS (PIOPS) SSD
- Critical business applications with sustained IOPS  
- Great for databases  
- **io1:** 4 GiB - 16 GiB, up to 64,000 PIOPS  
- **io2 Block Express:** 4 GiB - 64 TiB, up to 256,000 PIOPS  

### Hard Disk Drive (HDD)
Cannot be a boot volume  
125 GiB to 16 TiB  

**Throughput Optimized HDD (st1):**  
- Big Data, Warehouses, Logs  
- Max throughput 500 MiB/s  

**Cold HDD (sc1):**  
- Lowest cost  
- Max throughput 250 MiB/s  

---

## Lesson 6: EBS Multi-Attach - io1/io2 family

- Attach the same EBS volume to multiple EC2 instances in the same AZ  
- Each instance has full read and write permissions  
- Up to 16 instances  
- Must use cluster-aware file system  
- Use case: clustered Linux apps (ex: Teradata)  

---

## Lesson 7: EBS Encryption

When you create an encrypted EBS volume, you get:  
- Data at rest encrypted  
- Data in flight encrypted  
- All snapshots encrypted  
- All volumes from snapshots encrypted  
- Encryption handled transparently  
- Minimal performance impact  
- Uses KMS (AES-256)  

### How to encrypt an unencrypted EBS volume?
1. Create snapshot  
2. Encrypt snapshot  
3. Create new EBS volume from snapshot  
4. Attach encrypted volume  

---

## Lesson 8: Amazon EFS

- Managed NFS that can be mounted on many EC2  
- Multi-AZ  
- Highly available, scalable, expensive  
- Use case: CMS, web serving, data sharing, Wordpress  
- Linux only  
- Encryption using KMS  
- POSIX compliant  
- Automatically scales, pay-per-use  

### Performance & Storage Classes

**EFS Scale:**  
- 1000s of clients  
- 10 GB+/s throughput  
- Petabyte-scale  

**Performance Mode:**  
- General Purpose  
- Max I/O  

**Throughput Mode:**  
- Bursting  
- Provisioned  
- Elastic  

**Storage Classes:**  
- Standard  
- Infrequent Access (EFS-IA)  
- Archive  

**Availability:**  
- Standard (Multi-AZ)  
- One-Zone (cheaper)  

---

## Lesson 9: EBS vs EFS

### EBS:
- One instance (except io1/io2 multi-attach)  
- Locked to AZ  
- gp2: IO linked to size  
- gp3/io1: IO configurable independently  
- Migrate by snapshot → restore  
- Backups use IO  
- Root volumes deleted by default on termination  

### EFS:
- Mount 100s of instances across AZ  
- Great for shared storage (WordPress)  
- Linux only  
- Higher cost  
- Storage tiers  

**Remember: EFS vs EBS vs Instance Store**
