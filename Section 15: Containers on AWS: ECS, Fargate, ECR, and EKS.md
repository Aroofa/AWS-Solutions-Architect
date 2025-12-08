# Containers Overview: Docker, ECS, and EKS

## What is Docker?
Docker is a software development platform to deploy applications using **containers**.  

**Key points:**
- Containers are standardized units of software.
- Run the same way on any operating system.
- Predictable behavior → easier maintenance and deployment.
- Works with any language, OS, or technology.

**Use Cases:**
- Microservice architecture
- Lift-and-shift apps from on-premises to the cloud
- Running any containerized applications

---

## Docker Architecture
- **Server**: Could be EC2 or any server
- **Docker Daemon**: Manages containers
- **Containers**: Run apps like Java, Node.js, MySQL, etc.

**Docker Images Storage:**
- **Public**: Docker Hub
- **Private**: Amazon ECR (Elastic Container Registry)
  - ECR also has a public gallery

---

## Docker vs Virtual Machines
**Virtual Machine:**
- Infrastructure → Host OS → Hypervisor → Guest OS → Apps
- Fully isolated, secure, but resource-heavy

**Docker Container:**
- Infrastructure → Host OS → Docker Daemon → Containers
- Lightweight, share host resources, higher density
- Slightly less isolated but more efficient

---

## Getting Started with Docker
1. **Write a Dockerfile** → defines container structure
2. **Build Docker Image** → adds files to a base image
3. **Push Image** → store in Docker Hub or Amazon ECR
4. **Pull & Run Image** → runs as a Docker container

---

## Docker Management on AWS
- **Amazon ECS (Elastic Container Service)**: AWS native container management
- **Amazon EKS (Elastic Kubernetes Service)**: Managed Kubernetes
- **AWS Fargate**: Serverless container platform (works with ECS & EKS)
- **Amazon ECR**: Container image storage

**Key Integration Points:**
- ECS and EKS orchestrate Docker containers
- Fargate eliminates server management

---

**Next Steps:** Deep dive into ECS, EKS, and Fargate for container orchestration on AWS.

# Amazon ECS Overview

Amazon ECS (Elastic Container Service) is AWS’s managed service to run Docker containers.  

---

## ECS Launch Types

### 1. EC2 Launch Type
- ECS tasks run on **EC2 instances** that you provision and maintain.
- Each EC2 instance must run the **ECS Agent** to register with the ECS Cluster.
- ECS automatically places containers (tasks) on instances.
- You manage scaling and capacity of EC2 instances.

**Key Points:**
- Manual infrastructure management.
- EC2 instances required.
- ECS tasks are placed on EC2 automatically.

### 2. Fargate Launch Type
- ECS tasks are **serverless**—no EC2 instances to manage.
- Define ECS tasks using **task definitions**.
- AWS runs the tasks based on CPU & RAM requirements.
- Scaling = increase number of tasks.
- Preferred for exams because it's serverless and easier to manage.

---

## IAM Roles in ECS

### EC2 Launch Type
- **EC2 Instance Profile Role**:
  - Used by ECS Agent.
  - Makes API calls to ECS, CloudWatch Logs, ECR, Secrets Manager, SSM Parameter Store.

### Task Roles (ECS Task Roles)
- Applies to **both EC2 and Fargate** launch types.
- Assign **specific roles per ECS task** for fine-grained permissions.
  - Example:
    - Task A Role → S3 access
    - Task B Role → DynamoDB access
- Role is defined in the **task definition**.

**Distinction:**  
- EC2 Instance Profile Role → ECS Agent (infrastructure level)  
- ECS Task Role → ECS Task (application level)

---

## Load Balancer Integration
- ECS tasks can be exposed via HTTP/HTTPS using load balancers.
- **Application Load Balancer (ALB)**: recommended, works with EC2 & Fargate.
- **Network Load Balancer (NLB)**: for high throughput or PrivateLink.
- **Classic Load Balancer**: not recommended, cannot attach to Fargate.

---

## Data Persistence
- ECS tasks can use **Data Volumes** for persistent storage.
- **Amazon EFS** (Elastic File System):
  - Network file system compatible with EC2 & Fargate.
  - Multi-AZ shared storage for containers.
  - Ideal for serverless persistent storage.
- Recommended combo: **Fargate + EFS** for serverless ECS with persistent storage.

---

## Key Takeaways
- EC2 Launch Type → managed EC2 infrastructure.  
- Fargate Launch Type → serverless, tasks scale automatically.  
- Task Roles provide fine-grained access per ECS task.  
- ALB is preferred for exposing ECS tasks.  
- Use EFS for persistent, shared, multi-AZ container storage.  

# ECS Service Auto Scaling

Amazon ECS allows you to **scale your services** automatically based on metrics, instead of manually increasing or decreasing ECS tasks.

---

## Scaling ECS Tasks

- **Manual Scaling**: Increase/decrease the number of tasks manually.
- **Automatic Scaling**: Use **AWS Application Auto Scaling** to adjust tasks dynamically.

### Metrics for Auto Scaling
You can scale ECS services based on **three metrics**:
1. **CPU Utilization** – percentage of CPU used by the ECS Service.
2. **Memory Utilization** – RAM used by the ECS Service.
3. **ALB Request Count per Target** – number of requests per target from the Application Load Balancer.

### Types of Auto Scaling
1. **Target Tracking**
   - Tracks a specific metric target (e.g., keep CPU at 60%).
2. **Step Scaling**
   - Scales in steps based on thresholds.
3. **Scheduled Scaling**
   - Scales ECS Service at predictable times (e.g., before traffic spikes).

**Note:** Scaling ECS tasks **does not automatically scale EC2 instances** if using EC2 Launch Type.

---

## Scaling EC2 Instances in ECS (EC2 Launch Type)

When ECS tasks run on EC2 instances, you may need to scale the EC2 cluster itself:

### 1. Auto Scaling Group (ASG)
- Scale EC2 instances manually or based on metrics (CPU, memory).
- Adds/removes EC2 instances as needed.

### 2. ECS Cluster Capacity Provider (Recommended)
- Paired with an ASG.
- Automatically scales EC2 instances when ECS tasks lack CPU or memory.
- Smarter and preferred over ASG alone.

---

## Example Flow
1. ECS Service A has **2 tasks**.
2. CPU usage increases due to more users.
3. **CloudWatch Metric** monitors CPU usage.
4. **CloudWatch Alarm** triggers scaling action.
5. AWS Application Auto Scaling increases ECS **desired task count**.
6. If using EC2 Launch Type:
   - ECS Cluster Capacity Provider can add EC2 instances to the cluster if needed.

---

## Key Takeaways
- ECS Service Auto Scaling = scaling **tasks**.  
- EC2 Launch Type requires separate scaling for **EC2 instances**.  
- Fargate simplifies scaling (serverless, no EC2 management).  
- **Capacity Provider** is smarter than standalone ASG for EC2 clusters.  
- Metrics to remember: CPU, Memory, ALB Request Count per Target.  
- Scaling types: Target Tracking, Step Scaling, Scheduled Scaling.

# ECS Solution Architectures

Amazon ECS can be integrated with other AWS services to build **serverless and event-driven architectures**. Here are some common patterns:

---

## 1. ECS Tasks Triggered by EventBridge from S3

- **Scenario**: Users upload objects to S3 buckets.
- **Integration**: S3 → EventBridge → ECS Task (Fargate)
- **Flow**:
  1. S3 event triggers an **EventBridge rule**.
  2. ECS Task is launched in **Fargate**.
  3. ECS Task has a **Task Role** for permissions (e.g., access S3, write to DynamoDB).
  4. Task processes the S3 object and stores results in **DynamoDB**.

**Use case**: Serverless processing of uploaded files/images.

---

## 2. Scheduled ECS Tasks via EventBridge

- **Scenario**: Run ECS tasks periodically (e.g., every hour).
- **Integration**: EventBridge → ECS Task (Fargate)
- **Flow**:
  1. EventBridge rule set on a **schedule**.
  2. ECS Task is created at the scheduled time.
  3. ECS Task Role allows access to required resources (e.g., S3).
  4. Task performs batch processing (e.g., data aggregation, file processing).

**Use case**: Automated, periodic processing with fully serverless architecture.

---

## 3. ECS Tasks Processing Messages from SQS

- **Scenario**: ECS service processes queued messages.
- **Integration**: SQS → ECS Service (Tasks in Fargate)
- **Flow**:
  1. Messages sent to **SQS queue**.
  2. ECS Tasks **pull messages** from SQS.
  3. ECS Service Auto Scaling can scale tasks based on **queue length**.

**Use case**: Event-driven processing of queued workloads, scalable with auto-scaling.

---

## 4. EventBridge for ECS Lifecycle Events

- **Scenario**: React to ECS task state changes.
- **Integration**: ECS → EventBridge → SNS (or other targets)
- **Flow**:
  1. EventBridge monitors **ECS task lifecycle** (started, stopped).
  2. Rule triggers when tasks **stop or exit**.
  3. Notifications sent to **SNS** → Email to admins.

**Use case**: Monitoring and alerting on ECS cluster/container lifecycle events.

---

## Key Points

- ECS + **Fargate** = fully serverless container execution.
- **EventBridge** can trigger ECS tasks from:
  - S3 events
  - Scheduled events
  - ECS lifecycle events
- ECS Tasks should have **Task Roles** for resource permissions.
- ECS + **SQS** allows scalable, queue-driven architectures.
- ECS Service Auto Scaling can increase tasks based on **SQS queue length**.

# Amazon ECR (Elastic Container Registry)

## Overview
- Amazon ECR is AWS's **Docker image repository**.
- Used to **store, manage, and version Docker images**.
- Fully integrated with **Amazon ECS**.

---

## Types of Repositories
1. **Private Repository**
   - Accessible only to your AWS account(s).
2. **Public Repository**
   - Publish images to the **Amazon ECR Public Gallery**.

---

## Key Features
- Images are stored on **Amazon S3** behind the scenes.
- **Pulling Images**:
  - ECS EC2 instances pull images from ECR.

# Amazon EKS (Elastic Kubernetes Service)

## Overview
- Managed Kubernetes service on AWS.
- Used to deploy, scale, and manage **containerized applications** (usually Docker).
- Open-source, cloud-agnostic alternative to Amazon ECS.

---

## Launch Modes
1. **EC2 Launch Mode**
   - Worker nodes are EC2 instances.
   - Nodes run **EKS Pods** (equivalent to ECS tasks).
   - Nodes can be managed via **Auto Scaling Groups (ASG)**.
2. **Fargate Launch Mode**
   - Fully serverless; no nodes to manage.
   - Pods run directly on Fargate.
   - No infrastructure maintenance required.

---

## Use Cases
- Migrating Kubernetes workloads from on-premises or other clouds.
- Standardization on Kubernetes APIs.
- Serverless container deployment using Fargate.

---

## Node Types
1. **Managed Node Groups**
   - AWS manages EC2 nodes for you.
   - Integrated with Auto Scaling Groups.
   - Supports On-Demand and Spot instances.
2. **Self-Managed Nodes**
   - You create EC2 nodes manually.
   - Nodes registered to EKS cluster.
   - Can use Amazon EKS Optimized AMI or custom AMI.
3. **Fargate**
   - Serverless; no nodes visible.
   - Fully managed by AWS.

---

## Load Balancer Integration
- Private or public load balancers can expose services.
- Similar to ECS ALB/NLB integration.

---

## Storage Integration
- Attach volumes using **StorageClass manifests**.
- Supports **CSI-compliant drivers**.
- Compatible storage options:
  - Amazon EBS
  - Amazon EFS (**works with Fargate**)
  - Amazon FSx for Lustre
  - Amazon FSx for NetApp ONTAP

---

**Exam Tip:**  
- Pods = EKS equivalent of ECS tasks.  
- Fargate mode = serverless, no nodes to manage.  
- Managed Node Groups = AWS-managed EC2 nodes with ASG.  
- EFS is the only persistent storage that works with Fargate.

# AWS App Runner

## Overview
- Fully managed service to deploy **web applications** and **APIs**.
- No need to manage infrastructure (servers, containers, scaling).
- Supports **source code** or **Docker container images** as input.

---

## Key Features
- **Automatic build and deployment** from source code or container.
- **Autoscaling** based on traffic.
- **Load balancing** built-in.
- **High availability** across AWS infrastructure.
- **Encryption** for in-transit and at-rest data.
- **VPC connectivity** for secure access to databases, caches, or queues.

---

## Use Cases
- Rapid deployment of web apps or APIs.
- Microservices deployment without managing containers.
- Quick production rollout with best practices automatically applied.

---

**Exam Tip:**  
- App Runner = serverless web app deployment.  
- Handles scaling, load balancing, and deployment automatically.  
- Simple configuration: CPU, memory, autoscaling, health checks.

  - Requires an **IAM role** with proper permissions.
- **Security & Management**:
  - IAM controls access.
  - Image vulnerability scanning.
  - Versioning and tagging.
  - Lifecycle policies for image cleanup.

---

## Integration with ECS
- ECS clusters (EC2 or Fargate) can pull images directly from ECR.
- EC2 instances must have IAM permissions to access the repository.
- After pulling, containers start running on ECS.

---

**Exam Tip:** Anytime you see "storing Docker images" or "container repository" in AWS context → **think ECR**.

# AWS App2Container (A2C)

## Overview
- CLI tool for **migrating and modernizing** Java and .NET web applications.
- Performs **lift-and-shift migrations** from on-premises (bare metal, VMs) to AWS.
- **No code changes** required.
- Generates **Docker containers** and deployment artifacts automatically.

---

## Key Workflow
1. **Discover & Analyze**: CLI identifies apps ready for migration.
2. **Containerize**: Extracts apps and packages them as Docker images.
3. **Generate Deployment Artifacts**:
   - CloudFormation templates for compute, network, and ECS/EKS/App Runner tasks.
   - Optional CI/CD pipeline artifacts.
4. **Deploy**:
   - Docker images stored in **Amazon ECR**.
   - Can deploy to **ECS**, **EKS**, or **App Runner**.

---

## Use Cases
- Migrating legacy Java/.NET apps to AWS.
- Modernizing apps without rewriting code.
- Rapid cloud adoption with containerized apps.

---

**Exam Tip:**  
- A2C = Lift-and-shift **container migration tool** for Java/.NET.  
- Works with ECS, EKS, and App Runner.  
- Generates CloudFormation + ECR images automatically.
