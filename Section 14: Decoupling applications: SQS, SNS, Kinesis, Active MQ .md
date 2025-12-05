# Lesson 1: Introduction to Messaging 
## Why Integration Matters
When deploying multiple applications, they must **communicate and share data**. AWS provides different messaging patterns to achieve this reliably and at scale.

---

## Application Communication Models

### 1. **Synchronous Communication**
- Services talk **directly** to each other.
- Example:
  - Buying Service → Shipping Service
  - When a purchase happens, the buying service directly triggers the shipping service.
- **Pros:** Real-time response.
- **Cons:** If one service is slow or overloaded, the other is affected (tight coupling).

---

### 2. **Asynchronous (Event-Based) Communication**
- A **middleware/queue** sits between services.
- Buying service sends event → Queue stores event  
- Shipping service later pulls event from queue and processes it.
  
**Pros of Asynchronous Design**
- Services are **decoupled**
- Handles **traffic spikes** better
- More **fault-tolerant**
- Services can scale **independently**

---

## When synchronous becomes a problem
If there is a **sudden spike in requests**, like encoding 1,000 videos instead of 10:
- The processing service can get overwhelmed.
- Leads to failures, delays, outages.

→ This is when **asynchronous messaging is preferred**.

---

## AWS Messaging & Streaming Tools

| Service | Use Case | Key Concept |
|--------|----------|-------------|
| **SQS (Simple Queue Service)** | Decoupling using queues | Producers push messages, consumers process later |
| **SNS (Simple Notification Service)** | Pub/Sub messaging | One-to-many message fan-out |
| **Kinesis** | Real-time streaming & big data processing | Handles large volume streams at low latency |

---

## Architecture Benefits
Using SQS, SNS, or Kinesis allows:
- Independent scaling of services
- Reliability under heavy load
- Loose coupling between systems
- Event-driven design patterns

---

# Lesson 2: Amazon SQS - Standard Queues Overview
# 📨 Amazon SQS (Simple Queue Service) – Complete Study Notes

Amazon SQS is a **fully managed distributed message queuing service** that helps **decouple applications** for scalability and reliability.

---

## 🔹 What is SQS?

- Stands for **Simple Queue Service**
- Stores messages temporarily until processed
- Enables **asynchronous communication**
- Used to **decouple producers and consumers**

---

## 🏗 Core Concepts

| Component | Role |
|----------|------|
| **Queue** | Stores messages temporarily |
| **Producer** | Sends messages into the queue |
| **Consumer** | Retrieves and processes messages |
| **Message** | Task/data to be processed (order ID, video encode job, etc.) |

---

## 📨 Message Flow

1. **Producer → SendMessage → Queue**
2. Consumer polls queue using `ReceiveMessage`
3. Consumer processes the message
4. Consumer calls `DeleteMessage` to remove it from queue

> If message isn't deleted, it can be processed again by another consumer (at-least-once delivery).

---

## ✨ Features of SQS Standard Queue

| Feature | Details |
|--------|---------|
| **Throughput** | Unlimited messages per second |
| **Queue Size** | Unlimited number of messages |
| **Message Size** | Up to **1,024 KB (1MB)** each |
| **Retention Period** | Default **4 days**, max **14 days** |
| **Latency** | < 10 ms for send/receive |
| **Delivery** | At-least-once (duplicates possible) |
| **Ordering** | Best-effort ordering (out of order possible) |

---

## 🧠 When to Use SQS

- To buffer requests between application components
- When processing load is **uneven or unpredictable**
- When tasks must continue even if one system fails
- When scaling consumers at different rates

**Exam hint** → If you see **application decoupling**, think **SQS**.

---

## 🏗 Architecture Example: Order Processing

**Producer:** Web app  
**Queue:** SQS  
**Consumer:** EC2/Lambda/App pulls messages  

Flow:
- Receive order → send to SQS
- Worker processes order → writes to DB → deletes message

---

## 🔥 Scaling With SQS + Auto Scaling Group

1. Consumers run in an **Auto Scaling Group**
2. Set **CloudWatch alarm based on queue length**
   - Metric: `ApproximateNumberOfMessages`
3. Auto Scaling increases consumers when queue is large
4. System handles traffic spikes automatically

✔ Perfect pattern for high-volume workloads

---

## 🖥 Consumer Processing Options

Consumers can run on:

- EC2 instances
- On-prem servers
- AWS Lambda (serverless processing)

---

## 📺 Example Use Case: Video Processing System

| Component | Role |
|----------|------|
| Frontend | Uploads video → sends message to SQS |
| Backend Processor | Pulls messages → encodes video |
| Storage | Upload output to S3 |

Advantages:

- Frontend remains fast
- Backend scales separately
- Can use GPU instances only where needed
- Queue acts as buffer for high loads

---

## 🔒 Security & Encryption

| Security Feature | Details |
|------------------|---------|
| **In-flight encryption** | HTTPS |
| **At-rest encryption** | KMS supported |
| **Client-side encryption** | Optional, managed by user |
| **IAM policies** | Control access to actions |
| **SQS Access Policies** | Allow cross-account or service access (SNS/S3 → SQS) |

---

## Quick Exam Notes 📝

- **Unlimited throughput**, **1 MB max message size**
- Default retention = **4 days**, max = **14 days**
- **At-least-once delivery → duplicates possible**
- **Best-effort ordering**
- Use **SQS Standard** for high throughput
- Use with **Auto Scaling based on queue length**
- **Decouples producers and consumers**

---

# Lesson 3: SQS - Message Visibilty Timeout
- When a consumer retrieves a message from an SQS queue, the message becomes **temporarily invisible** to other consumers. This period is called the **Visibility Timeout**.

### How It Works
1. Consumer calls `ReceiveMessage`
2. Message becomes **invisible** to others for the visibility timeout duration
3. Consumer processes the message
4. When done → consumer must call **`DeleteMessage`**
5. If not deleted before timeout expires → message becomes **visible again** and may be reprocessed

### Default Behavior
- Default timeout: **30 seconds**
- Configurable: **0 seconds – 12 hours**
- If message isn't processed in time → it may be processed again → **duplicate processing** risk

### Extending Processing Time
If a consumer needs longer to process a message:

```bash
ChangeMessageVisibility

# Lesson 4: SQS - Long Polling
**Long polling** allows a consumer to wait for messages to arrive if the queue is empty.  
Instead of returning an empty response immediately, SQS holds the request open until a message becomes available or the wait time expires.

### Why Use Long Polling?
- Reduces number of `ReceiveMessage` API calls
- Lowers cost
- Reduces latency (message delivered as soon as it arrives)
- More efficient than short polling

### Configuration
- Wait time range: **1–20 seconds**
- Enable via:
  - Queue settings (default wait time)
  - `ReceiveMessage` API using `WaitTimeSeconds`

### Best Practice
✔ **Use long polling instead of short polling whenever possible.**

Example (pseudo):

```bash
ReceiveMessage --WaitTimeSeconds 20
```
This extends the invisibility window without returning the message to the queue.

### Key Points for Exams
- Prevent duplicate processing → use ChangeMessageVisibility
- Too short timeout → messages may reappear and get processed multiple times
- Too long timeout → slow recovery if a consumer crashes
- Set timeout based on expected processing duration

# Lesson 5: SQS - FIFO Queues 
# Amazon SQS FIFO Queue

## 📌 What is FIFO?

FIFO stands for **First-In-First-Out** – messages are received **in the exact order they are sent**.

A **Standard SQS queue does NOT guarantee ordering**, but a FIFO queue does.

---

## Key Features of FIFO SQS

- Maintains **strict message ordering**
- Supports **Exactly-Once Processing**
- Prevents duplicates using **Deduplication ID**
- Lower throughput than Standard SQS
- Queue name **must end with `.fifo`**

---

## Throughput Limits

| Queue Type | Messages per second |
|-----------|---------------------|
| FIFO (without batching) | ~300 msg/s |
| FIFO (with batching) | ~3000 msg/s |

---

## Deduplication

To avoid duplicate messages:

- Provide a **`DeduplicationId`** when sending
- If the same ID is reused within **5 minutes**, duplicates are discarded

Example:

```json
{
  "MessageBody": "Hello World 1",
  "MessageGroupId": "demo",
  "DeduplicationId": "1"
}
```
## Message Group ID
- Required in FIFO queues
- Ensures messages with the same group ID are processed in order
Multiple consumers can process different message groups in parallel.

## Creating a FIFO Queue

1. Go to **Amazon SQS Console**
2. Click **Create Queue**
3. Select **FIFO Queue**
4. Queue name **must end with `.fifo`**
5. (Optional) Enable **Content-based Deduplication**
6. Click **Create Queue**

---

## Sending Messages Example

```json
{
  "MessageBody": "Hello World 1",
  "MessageGroupId": "demo",
  "DeduplicationId": "1"
}
```
## Receiving Messages (FIFO Behavior)

Messages will be received **in the same order they were sent**:

- Hello World 1  
- Hello World 2  
- Hello World 3  
- Hello World 4  

After processing → **Delete messages from the queue**

---

## When to Use FIFO vs Standard Queue

| Use Case                          | Recommended Queue |
|----------------------------------|-------------------|
| Order-sensitive workloads        | **FIFO**          |
| Banking / transactions / orders  | **FIFO**          |
| Simple background jobs           | Standard          |
| Very high throughput workloads   | Standard          |

---

# Lesson 6: SQS + Auto Scaling Group
## Using SQS with Auto Scaling Groups

Amazon SQS can be combined with Auto Scaling Groups (ASG) to process messages efficiently and handle traffic spikes.

---

### Architecture

1. **Frontend Web Application**: Receives user requests  
2. **SQS Queue**: All requests are sent as messages into the queue  
3. **Backend Workers / Auto Scaling Group**: EC2 instances poll the SQS queue, process messages, and write results (e.g., to a database)

---

### Scaling Pattern

- **Metric**: Use CloudWatch metric `ApproximateNumberOfMessages` (queue length)  
- **Alarm**: Set a threshold, e.g., 1,000 messages in the queue → triggers a scaling action  
- **Scaling**: ASG adds or removes EC2 instances based on the alarm, ensuring fast processing

---

### Benefits

- **Decouples application tiers**  
- **Handles sudden traffic spikes**  
- **Ensures durable message storage**  
- **Prevents database overload**  
- **Supports horizontal scaling of consumers**

---

### Common Use Cases / Exam Patterns

1. **Database buffering**: Instead of writing directly to RDS / DynamoDB, enqueue transactions in SQS → ASG workers dequeue → write to database → delete from queue  
2. **Application tier decoupling**: Frontend sends requests to SQS → backend processes asynchronously  
3. **High-volume workloads**: Any scenario where traffic spikes or system load is unpredictable

> **Exam Tip**: Look for keywords like *decouple systems, asynchronous processing, traffic spikes, buffer messages, scalable architecture* → SQS is usually the right answer.


# Lesson 7: Amazon Simple Notification Service (AWS SNS)

Amazon SNS is a **Pub/Sub (Publish-Subscribe) service** that allows one message to be sent to many receivers.

---

## Why Use SNS?

Instead of directly sending a message to multiple services (email, shipping, fraud service, SQS queue), you can use:

- **SNS Topic**: Producer publishes a message to a topic
- **Subscribers**: Multiple receivers (services or endpoints) get the message automatically

This decouples the producer from the consumers and simplifies integration.

---

## Subscribers

SNS subscribers can include:

- **Email notifications**
- **SMS / mobile notifications**
- **HTTP / HTTPS endpoints**
- **AWS Services**: SQS, Lambda, Kinesis Data Firehose, Redshift

SNS can receive notifications from AWS services like:

- CloudWatch Alarms  
- Auto Scaling Groups  
- CloudFormation state changes  
- Budgets  
- S3 buckets, DMS, Lambda, DynamoDB, RDS events

> SNS automatically routes messages to subscribers.

---

## Limits Overview

- **Subscriptions per topic**: Up to 12,000,000+  
- **Topics per account**: Up to 100,000 (can request increase)  
- *Note*: You are not typically tested on exact limits

---

## How It Works

1. **Create a Topic**  
2. **Create Subscriptions** (one or many

# Lesson 8: SNS and SQS - Fan Out Pattern
# SNS + SQS Fan-Out Pattern

The **Fan-Out Pattern** allows a single message sent to an SNS topic to be delivered to **multiple SQS queues** or other subscribers.  

This pattern is highly useful for **decoupling** and **scalable architectures**.

---

## How It Works

1. Producer sends a message to an **SNS topic**.  
2. Multiple **SQS queues** subscribe to the topic.  
3. Each SQS queue receives the message independently.  
4. This allows for **delayed processing**, **data persistence**, and **automatic retries**.

> Benefits: fully decoupled, no data loss, easy to add more subscribers over time.

---

## Example Use Cases

### Multi-Service Processing
- Buying service sends a message to **SNS topic**  
- Subscribers:
  - Fraud service → SQS queue  
  - Shipping service → SQS queue  

### S3 Event Notifications
- S3 bucket events → SNS topic → multiple SQS queues  
- Useful because S3 event rules allow **only one SQS target per rule**  
- Fan-out allows multiple destinations for a single event  

### Integration with Kinesis Data Firehose
- SNS topic → Kinesis Data Firehose → S3 bucket or other destinations  
- Extensible for storing or processing messages  

---

## SNS FIFO Topics + SQS FIFO Queues
- **FIFO guarantees** ordering and deduplication:
  - Ordering by **Message Group ID**  
  - Deduplication via **Deduplication ID** or **content-based deduplication**
- **Subscribers** must be **SQS FIFO queues**  
- **Throughput limits** same as SQS FIFO queues  

---

## Message Filtering
- SNS supports **subscription filter policies** in JSON
- Default: subscriptions receive **all messages**  
- Example:  
  ```json
  {
    "state": ["Placed"]
  }
  ```
## Message Filtering in SNS

- Only messages with `"state": "Placed"` go to this subscription.  
- Another subscription can filter `"state": "Canceled"`.  

### Filter Application
Filters can be applied to:
- SQS queues
- Emails
- Lambda functions

> Message filtering + fan-out + FIFO gives **flexibility**, **scalability**, and **precise control** over message delivery.

---

## Summary

- Fan-out pattern simplifies sending a single message to multiple destinations.  
- Fully decoupled architecture → reduces errors and increases reliability.  
- Supports SQS queues, Lambda, email, mobile, Kinesis Firehose.  
- Can be combined with FIFO, deduplication, and filter policies for complex workflows.

# Lesson 9: Amazon Kinesis Data Streams
# Amazon Kinesis Data Streams

Amazon Kinesis Data Streams is a service used to **collect and store streaming data in real time**.  
> Keyword for the exam: **real-time**

---

## Real-Time Data Examples
- Click streams from a website
- IoT devices (e.g., connected bicycles)
- Metrics and logs from servers

---

## Data Flow

1. **Producers**: Send data into Kinesis Data Streams
   - Applications (custom code)
   - Kinesis Agent for server logs and metrics
2. **Consumers**: Read and process data
   - Applications (custom code)
   - Lambda functions
   - Amazon Kinesis Data Firehose
   - Analytics services (e.g., Managed Service for Apache Flink)

---

## Features
- **Data Retention**: Up to 365 days
  - Allows reprocessing and replaying of data
  - Data cannot be deleted manually; only expires after retention
- **Data Size**: Up to 1 MB per record
- **Ordering**: Maintained for records with the same **Partition Key**
- **Security**:
  - At-rest encryption using KMS
  - In-flight encryption using HTTPS
- **Libraries for optimization**:
  - Producer → **Kinesis Producer Library (KPL)**
  - Consumer → **Kinesis Client Library (KCL)**

---

## Capacity Modes

### 1. Provisioned Mode
- You choose the number of **shards**
- **Shard throughput**:
  - Inbound: 1 MB/sec or 1,000 records/sec per shard
  - Outbound: 2 MB/sec per shard
- **Scaling**:
  - Manually add or remove shards based on throughput
- **Billing**: Pay per shard per hour

### 2. On-Demand Mode
- No need to provision capacity
- Default capacity: 4,000 records/sec or 4 MB/sec
- Automatically scales based on throughput over the past 30 days
- **Billing**: Pay per stream per hour + amount of data in/out

---

## Key Takeaways
- Kinesis Data Streams is ideal for **real-time streaming data**
- Provides durable, replayable data storage
- Supports high-throughput applications with **Provisioned** or **On-Demand** scaling

# Lesson 10: Amazon Data Firehouse
# Amazon Data Firehose

Amazon Data Firehose is a service to **send streaming data from sources into target destinations**.  
> Keyword for exams: **near real-time**

---

## Data Flow

1. **Producers**
   - Applications, clients, or custom SDK integrations
   - Kinesis Agents
   - AWS services (e.g., Kinesis Data Streams, CloudWatch Logs/Events, IoT)

2. **Data Transformation**
   - Optional Lambda function for data transformation (e.g., format conversion)

3. **Buffering**
   - Firehose accumulates data in a buffer before writing in batches
   - Buffer can be based on **size** or **time**

4. **Destinations**
   - **AWS Services**: S3, Redshift, OpenSearch
   - **Third-party Partners**: Datadog, Splunk, New Relic, MongoDB
   - **Custom HTTP Endpoints**: For unsupported destinations
   - Optional backup to S3 for failed data

---

## Features

- **Fully managed and serverless**
- **Automatic scaling**
- **Near real-time** delivery (due to buffering)
- Supports multiple input formats:
  - CSV, JSON, Parquet, Avro, text, binary
- Data conversion and compression:
  - Convert to Parquet or ORC
  - Compression: gzip, snappy
- **Custom transformations** via AWS Lambda (e.g., CSV → JSON)

---

## Comparison: Kinesis Data Streams vs Data Firehose

| Feature                     | Kinesis Data Streams                  | Amazon Data Firehose            |
|-------------------------------|--------------------------------------|--------------------------------|
| Type                         | Streaming data collection            | Streaming data delivery        |
| Real-Time                     | ✅ True real-time                     | ⚡ Near real-time               |
| Producer/Consumer Code        | Required                              | Not required (managed)         |
| Storage                       | Up to 1 year                          | None                           |
| Replay Capability             | ✅ Yes                                 | ❌ No                           |
| Scaling                       | Manual / On-demand                     | Automatic                      |
| Destinations                  | Custom via consumers                  | S3, Redshift, OpenSearch, Third-party, HTTP |

# Lesson 11: SQS vs SNS vs Kinesis
# SQS vs SNS vs Kinesis Overview

Understanding the differences between **SQS**, **SNS**, and **Kinesis** is crucial for AWS architecture and exam prep.

---

## Amazon SQS (Simple Queue Service)

- **Model:** Pull-based, consumers request messages from the queue
- **Message Processing:** Consumer must delete messages after processing
- **Scalability:** Managed service; scales automatically to hundreds of thousands of messages
- **Ordering:** Only guaranteed with **FIFO queues**
- **Delay:** Individual message delay is supported (e.g., 30 seconds)
- **Use Case:** Decoupling services, asynchronous processing, buffering workloads

---

## Amazon SNS (Simple Notification Service)

- **Model:** Push-based, Pub/Sub model
- **Subscribers:** Pushes a copy of each message to all subscribers
- **Scalability:** Up to 12,500,000 subscribers per topic; scales automatically
- **Persistence:** Not persistent — undelivered messages can be lost
- **Integration:** Can be combined with SQS (fan-out pattern) or Lambda, email, mobile, Kinesis Firehose
- **Use Case:** Broadcast messages to multiple systems/services

---

## Amazon Kinesis Data Streams

- **Model:** Streaming data collection
- **Consumption Modes:**
  - **Standard:** Consumers pull data (2 MB/sec per shard)
  - **Enhanced Fan-Out:** Kinesis pushes data to consumers (2 MB/sec per shard per consumer)
- **Persistence:** Data stored and replayable
- **Ordering:** Guaranteed at the **shard level**
- **Scalability:** Number of shards determines throughput; can scale manually (Provisioned) or automatically (On-Demand)
- **Retention:** 1–365 days
- **Use Case:** Real-time big data, analytics, ETL pipelines

---

## Quick Comparison Table

| Feature                  | SQS                  | SNS                         | Kinesis Data Streams                 |
|---------------------------|--------------------|----------------------------|------------------------------------|
| Data Model               | Pull (Queue)        | Push (Pub/Sub)             | Pull/Push (Streaming)              |
| Persistence              | ✅ Durable           | ❌ Not persistent          | ✅ Durable, replayable             |
| Ordering Guarantee       | FIFO only           | ❌ No                      | ✅ Shard-level                     |
| Throughput Provisioning  | Auto-scaled         | Auto-scaled                | Provisioned / On-Demand            |
| Real-Time / Near Real-Time | ❌ Asynchronous    | ⚡ Near real-time           | ✅ Real-time                        |
| Use Case                 | Decoupling, buffering | Broadcast to multiple consumers | Real-time analytics, ETL         |

# Lesson 12: Amazon MQ
# Amazon MQ Overview

Amazon MQ is a **managed message broker service** for migrating traditional on-premises messaging applications to the cloud without re-engineering.

---

## Why Amazon MQ?

- SQS and SNS are **cloud-native** with AWS proprietary APIs.
- Traditional applications often use **open protocols**:
  - MQTT
  - AMQP
  - STOMP
  - OpenWire
  - WSS
- When migrating to AWS, you may want to keep these protocols instead of rewriting your application for SQS/SNS.
- Amazon MQ provides managed **RabbitMQ** and **ActiveMQ** brokers in the cloud.

---

## Key Points

- Amazon MQ supports **queues** (like SQS) and **topics** (like SNS) in a single broker.
- **Scaling** is more limited compared to SQS/SNS; it runs on servers.
- Supports **high availability (HA)** via multi-AZ setup with failover.
- Multi-AZ failover requires **Amazon EFS** as backend storage to ensure data is shared across AZs.
- Clients can continue operations during failover with no data loss.

---

## High Availability Architecture

1. Deploy broker in **two AZs** (e.g., us-east-1a active, us-east-1b standby).
2. Both AZs mount the same **Amazon EFS** backend storage.
3. On failover:
   - Standby broker takes over.
   - Data is retained and consistent.
4. Ensures safe and reliable messaging during failures.

---

## Summary

- Managed RabbitMQ and ActiveMQ in the cloud.
- Ideal for **legacy applications using open protocols**.
- Supports **queues and topics**.
- Multi-AZ high availability ensures **data safety** during failover.
- Does **not scale as infinitely** as SQS/SNS.
