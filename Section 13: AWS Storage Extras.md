# Lesson 1: AWS Snowball

AWS Snowball is a **highly secure and portable device** that allows you to **collect, process, and migrate data** at the edge and into AWS.

## Use Cases
1. **Data Migration**
   - Ideal for migrating **large datasets (petabytes)** when network transfer is slow or expensive.
   - Example: Transferring 100 TB over a **1 Gbps connection** would take ~12 days.
   - Challenges with network transfer:
     - Limited connectivity or bandwidth
     - High network costs
     - Shared bandwidth with other applications
     - Unstable connections
   - **Solution:** Snowball device
     1. AWS ships a physical Snowball device to your location.
     2. You load your data onto the device.
     3. Ship it back to AWS.
     4. Data is imported into **Amazon S3**.

2. **Edge Computing**
   - Process data **where it is generated**, especially in locations with **limited or no internet**.
   - Examples:
     - Trucks on the road
     - Ships at sea
     - Mining stations
   - **Compute Optimized Snowball** can run:
     - EC2 instances
     - Lambda functions
     - Machine learning tasks
     - Media transcoding
   - Once processed, data can be sent back to AWS.

## Snowball Device Types
| Device Type | Storage | Purpose |
|-------------|---------|---------|
| Edge Storage Optimized | 210 TB | Bulk data migration |
| Edge Compute Optimized | 28 TB  | Edge computing and processing |

## Summary
- **Snowball** solves problems related to:
  - Slow or limited network bandwidth for large-scale data transfers
  - Processing data at the edge where connectivity is limited
- Supports **data migration** and **edge computing**.
- Edge devices allow **pre-processing**, **machine learning**, or **media transcoding** before sending data to AWS.

---

# Lesson 2: Snowball into Amazon Glacier Scenario

- **Problem:** Snowball **cannot import data directly into Glacier**.
- **Solution:** Use **Amazon S3 as an intermediary**.
  1. Import data from Snowball into **S3**.
  2. Create an **S3 Lifecycle Policy** to transition the objects into **Amazon Glacier**.

> **Key Exam Tip:** Always remember that Snowball → S3 → Glacier via lifecycle policy.

---
# lesson 3: Amazon FSx Overview

Amazon FSx allows you to launch **third-party high-performance file systems** on AWS as a fully managed service.  
Think of it like **RDS for file systems**.

You should know **four main FSx types** for the exam:

---

## 1. FSx for Windows File Server
- Fully managed Windows File Server share.
- Supports **SMB protocol** and **NTFS**.
- Integrates with **Active Directory**, uses **ACLs** and **user quotas**.
- Can also be **mounted on Linux EC2 instances**.
- Supports **DFS integration** for on-premises Windows File Servers.
- **Performance:** Tens of GB/s, millions of IOPS, hundreds of PB.
- **Storage options:** SSD (low latency, IOPS-intensive) or HDD (broad workloads, cheaper).
- Can be **Multi-AZ**, accessible from on-premises privately.
- Daily backups to **Amazon S3**.

---

## 2. FSx for Lustre
- **High-performance distributed file system** for large-scale computing.
- Used for **Machine Learning, HPC, Video Processing, Financial Modeling**.
- **Performance:** Hundreds of GB/s, millions of IOPS, sub-ms latency.
- **Storage options:** SSD (low latency, IOPS-heavy), HDD (throughput-intensive).
- Integrates with **Amazon S3** for reading/writing.
- Accessible from on-premises via **VPN or Direct Connect**.

### Deployment options:
1. **Scratch File System**
   - Temporary storage, **no replication**.
   - Extremely high performance (6x persistent).
   - Use case: Short-term processing.
2. **Persistent File System**
   - Long-term storage, data **replicated within the same AZ**.
   - Use case: Long-term sensitive data.

---

## 3. FSx for NetApp ONTAP
- Managed **NetApp ONTAP file system** on AWS.
- Protocols: **NFS, SMB, iSCSI**.
- Compatible with **Linux, Windows, macOS, VMware Cloud, WorkSpaces, AppStream, EC2, ECS, EKS**.
- Features:
  - Auto-scaling storage
  - Snapshots and replication
  - Compression and de-duplication
  - Point-in-time instantaneous cloning (great for testing workloads)

---

## 4. FSx for OpenZFS
- Managed **OpenZFS file system** on AWS.
- Protocols: **NFS**.
- Use case: Migrate existing **ZFS workloads** to AWS.
- Features:
  - High performance: up to 1M IOPS, <0.5 ms latency
  - Snapshots, compression
  - No de-duplication
  - Supports **point-in-time cloning**

---

**Key Exam Tip:** Pick the FSx type based on:
- Windows workloads → FSx for Windows File Server  
- High-performance computing → FSx for Lustre  
- NetApp ONTAP migration → FSx for NetApp ONTAP  
- ZFS migration → FSx for OpenZFS

---

# lesson 4: AWS Storage Gateway Overview

AWS Storage Gateway is a **hybrid cloud service** that connects **on-premises infrastructure** with **AWS cloud storage**.  
It allows you to **extend, back up, or migrate storage** from your data center to AWS.

---

## Hybrid Cloud Use Cases
- Gradual cloud migration
- Security or compliance requirements
- Elastic workloads in cloud, core workloads on-premises
- Disaster recovery
- Low-latency access to frequently used data

---

## Storage Types in AWS
- **Block Storage:** Amazon EBS, EC2 Instance Store
- **File Storage:** Amazon EFS, Amazon FSx
- **Object Storage:** Amazon S3, Amazon Glacier

---

## Storage Gateway Types

### 1. S3 File Gateway
- Provides **file-based access (NFS/SMB)** to **S3 buckets**.
- Caches most recently used files on-premises for low-latency access.
- Supports multiple S3 storage classes (**Standard, Standard-IA, One Zone-IA, Intelligent-Tiering**; Glacier via lifecycle policy).
- **Active Directory integration** for SMB authentication.
- Use Case: Expose S3 objects as a **local file share**.

### 2. Volume Gateway
- Provides **block-level access (iSCSI)** to cloud-backed volumes.
- **Types:**
  - **Cached Volumes:** Store frequently accessed data on-premises, rest in S3.
  - **Stored Volumes:** Store full dataset on-premises; scheduled backup to S3.
- **Backups:** EBS snapshots in S3 for restoration.
- Use Case: Backup or extend on-premises block storage to AWS.

### 3. Tape Gateway
- Provides **virtual tape library (VTL)** backed by **S3 and Glacier**.
- Supports **iSCSI protocol** and integrates with existing tape-based backup software.
- Use Case: Replace physical tape backups with cloud storage.

---

## Summary Diagram
- **On-premises:** Storage Gateway VM installed
- **Cloud:** Storage Gateway service connected to S3 (and optionally Glacier)
- **Data Flow:**
  1. File Gateway → S3 → Lifecycle to Glacier if needed
  2. Volume Gateway → S3 → EBS snapshots
  3. Tape Gateway → S3 → Glacier/Deep Archive

---

**Exam Tip:**  
- **File Gateway:** NFS/SMB file share backed by S3  
- **Volume Gateway:** iSCSI volumes backed by S3/EBS snapshots  
- **Tape Gateway:** Backup tape replacement with

---

# lesson 5: AWS Transfer Family Overview

AWS Transfer Family is a **fully managed service** to transfer files in and out of **Amazon S3 or Amazon EFS** using standard file transfer protocols instead of native APIs.

---

## Supported Protocols
- **FTP:** Unencrypted File Transfer Protocol  
- **FTPS:** File Transfer Protocol over SSL/TLS (encrypted in transit)  
- **SFTP:** Secure File Transfer Protocol (encrypted in transit)  

> Key point: FTP is unencrypted, while FTPS and SFTP are encrypted.

---

## Features
- **Fully managed:** AWS handles scalability, reliability, and high availability.  
- **Integrates with existing authentication systems:**  
  - Microsoft Active Directory  
  - LDAP  
  - Okta  
  - Amazon Cognito  
  - Custom authentication sources  
- **IAM integration:** Uses IAM roles to access S3 or EFS transparently.  
- **Scalable & reliable:** Multiple endpoints can be provisioned.

---

## Pricing
- Pay **per provisioned endpoint per hour**  
- Pay per **GB of data transferred** in/out  

---

## Use Cases
- Provide **FTP/SFTP/FTPS access** to S3 or EFS  
- Share files or public datasets  
- Integrate with CRM, ERP, or legacy systems that require standard FTP protocols  

---

## Architecture Overview
1. Users connect to **Transfer Family endpoints** (FTP/SFTP/FTPS).  
2. Endpoints can optionally use a **custom hostname via Route 53**.  
3. AWS assumes an **IAM role** to read/write files from S3 or EFS.  
4. Users can be authenticated via external identity providers for **security and compliance**.

---

**Exam Tip:**  
- Use **AWS Transfer Family** when you need **standard file transfer protocols** for S3 or EFS without using AWS-native APIs.

---
# lesson 6: AWS DataSync Overview

AWS DataSync is a **managed service** for **moving and synchronizing large amounts of data** between on-premises, other cloud providers, and AWS storage services.

---

## Key Features
- **Supported destinations:**  
  - Amazon S3 (all storage classes including Glacier)  
  - Amazon EFS  
  - Amazon FSx
- **Supported sources:**  
  - On-premises NFS or SMB servers  
  - Other cloud storage locations  
  - AWS storage services (for intra-cloud sync)
- **Preserves file metadata and permissions:**  
  - NFS POSIX file system permissions  
  - SMB file system permissions  

---

## How It Works
1. **Install DataSync agent** on-premises or on Snowcone (pre-installed).  
2. Connect the agent to the source server (NFS/SMB).  
3. DataSync transfers files to AWS storage (S3, EFS, FSx).  
4. Can synchronize data **one-way or two-way** between source and destination.  
5. **Scheduled tasks:** hourly, daily, or weekly (not continuous).  
6. Optional **bandwidth throttling** to avoid netwo

---

# lesson 7: AWS Storage & Data Transfer Services – Summary

AWS offers **many storage options**, each with specific use cases. This guide summarizes object, block, file storage, and hybrid/offline transfer options.

---

## Object Storage

### Amazon S3
- **Type:** Object storage
- **Use Case:** Store and retrieve any amount of data via API
- **Key Notes:** Highly durable, integrates with AWS services

### Amazon S3 Glacier / Glacier Deep Archive
- **Type:** Object archival storage
- **Use Case:** Long-term storage with lower cost
- **Access:** Retrieval takes minutes to hours

---

## Block Storage

### Amazon EBS (Elastic Block Store)
- **Type:** Block storage for **single EC2 instance**
- **Use Case:** Persistent storage with high performance
- **Volume Types:** GP3, IO1, IO2 (high IOPS), etc.
- **Features:** Multi-Attach (for IO1/IO2)

### EC2 Instance Store
- **Type:** Temporary physical storage **attached to EC2**
- **Use Case:** High-performance local storage
- **Limitation:** Data lost on instance stop/terminate

---

## File Storage

### Amazon EFS (Elastic File System)
- **Type:** Network file system (NFS)
- **Use Case:** Shared storage across multiple Linux EC2 instances
- **Features:** POSIX-compliant, scalable, multi-AZ access

### Amazon FSx
- **FSx for Windows File Server:** SMB protocol, Active Directory integration
- **FSx for Lustre:** High-performance Linux file system, HPC, ML, S3 integration
- **FSx for NetApp ONTAP:** Broad OS compatibility, snapshots, deduplication, cloning
- **FSx for OpenZFS:** Managed ZFS, NFS protocol, high IOPS, snapshots, cloning

---

## Hybrid & On-Premises Storage Bridging

### AWS Storage Gateway
- **S3 / FSx File Gateway:** NFS/SMB access to cloud file storage
- **Volume Gateway:** iSCSI volumes, backed up to AWS
- **Tape Gateway:** Virtual tape library for backup to S3/Glacier

### AWS Transfer Family
- **Protocols:** FTP, FTPS, SFTP
- **Use Case:** Managed interface to S3 or EFS without using APIs

### AWS DataSync
- **Use Case:** Scheduled sync of large datasets between on-premises and AWS or between AWS services
- **Features:** Preserves metadata & permissions, requires agent for on-prem sources
- **Tip:** Snowcone can be used with pre-installed DataSync agent if network bandwidth is limited

---

## Offline / Physical Data Transfer

### AWS Snow Family
- **Snowcone:** Small, portable device, pre-installed DataSync agent
- **Snowball:** Edge storage or compute device for larger datasets
- **Snowmobile:** Massive physical transfer for exabyte-scale data

---

## Summary Tips for Exam
- **Object storage:** S3 / Glacier  
- **Block storage:** EBS / Instance Store  
- **File storage:** EFS / FSx variants  
- **Hybrid access / bridging:** Storage Gateway / Transfer Family / DataSync  
- **Offline large data transfer:** Snowcone / Snowball / Snowmobile  

> Choose the right service based on **workload type, performance needs, OS compatibility, and data transfer constraints**.

