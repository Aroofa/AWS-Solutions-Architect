## Lesson 1: S3 Lifecycle Rules & Analytics
**Purpose:** Automate transitions of objects between storage classes and deletions.

### Storage Classes & Transitions
- Standard → Standard-IA → Intelligent Tiering → One-Zone IA → Glacier / Deep Archive
- Use cases:
  - Frequently accessed → Standard
  - Infrequently accessed → Standard-IA
  - Archival → Glacier / Deep Archive

### Lifecycle Rules Components
1. **Transition Action:** Move objects between storage classes.
   - Example: Standard → Standard-IA after 60 days, Glacier after 6 months.
2. **Expiration Action:** Delete objects automatically.
   - Logs: delete after 365 days.
   - Delete old versions (with versioning enabled).
   - Delete incomplete multipart uploads (> 2 weeks old).
3. **Scope Options:**
   - Entire bucket
   - Specific prefixes (folders)
   - Specific tags (e.g., `department:finance`)

### Example Scenario
- **EC2 App Thumbnails & Source Images**
  - Source images: Standard → Glacier after 60 days.
  - Thumbnails: One-Zone IA → expire after 60 days.
- **Versioned Objects for Recovery**
  - Immediate recovery within 30 days.
  - Non-current versions → Standard-IA → Glacier Deep Archive.

### Analytics
- **S3 Analytics:** Generates CSV reports with recommendations.
- Helps optimize transition timing from Standard → Standard-IA.
- Updates daily, visible after 24–48 hours.

---

## Lesson 2: S3 Requester Pays
- By default: **Bucket owner pays storage & network costs.**
- **Requester Pays:** Network cost is billed to requester.
- Use case: Sharing large datasets.
- Requirements: Requester **must be authenticated**.

---

## Lesson 3: S3 Event Notifications
**Purpose:** React automatically to S3 events (create, delete, restore, replicate).

### Destinations
- SNS topic
- SQS queue
- Lambda function
- EventBridge (advanced filtering & multi-service integration)

### Requirements
- Resource-based access policies (SNS/SQS/Lambda)
- Not IAM roles for S3.
- EventBridge allows:
  - Advanced filtering (metadata, object name, size)
  - Multiple destinations
  - Event replay & archival

---

## Lesson 4: S3 Performance
### Baseline Performance
- Latency: 100–200 ms (first byte)
- Requests per second **per prefix:**
  - PUT/COPY/POST/DELETE → 3,500/sec
  - GET/HEAD → 5,500/sec
- Multiple prefixes → scale linearly

### Optimization Techniques
1. **Multipart Upload**
   - Recommended > 100 MB, required > 5 GB
   - Parallelizes uploads → maximizes bandwidth
2. **Transfer Acceleration**
   - Uses edge locations → faster global uploads/downloads
3. **Byte Range Fetches**
   - Parallelizes GET requests
   - Partial file retrieval
   - Retry only specific byte ranges

---

## Lesson 5: S3 Batch Operations
- Perform bulk operations on multiple objects with a **single request**.
- Use cases:
  - Modify metadata or ACLs
  - Copy objects between buckets
  - Encrypt unencrypted objects
  - Restore objects from Glacier
  - Invoke Lambda function for custom action
- **Job = Object list + Action + Optional parameters**
- **Filtered object list:** S3 Inventory + Athena

---

## Lesson 6: S3 Storage Lens
**Purpose:** Analyze, optimize, and protect S3 storage at scale.

### Features
- Metrics aggregation by:
  - Organization / Account / Region / Bucket / Prefix
- Default or custom dashboards
- Export metrics as CSV or Parquet
- Free metrics: ~28 usage metrics (14 days history)
- Advanced metrics (paid): activity, cost optimization, data protection, status codes (15 months history)

### Metric Categories
1. **Summary Metrics:** total storage, object count, average size
2. **Cost Optimization:** non-current versions, incomplete uploads
3. **Data Protection:** versioning, MFA delete, KMS encryption
4. **Access Management:** bucket ownership settings
5. **Event Metrics:** S3 event notification setup
6. **Performance:** transfer acceleration usage
7. **Activity:** GET/PUT requests, bytes transferred
8. **HTTP Status Codes:** 200, 403, etc.

---

## Key Exam Tips
- Lifecycle rules: know **transition vs expiration** and **prefix/tag filters**
- Requester Pays: network billed to requester
- Event notifications: **SNS, SQS, Lambda, EventBridge** & resource policies
- Performance: **multipart upload, transfer acceleration, byte range fetch**
- Batch operations: **S3 Inventory + Athena** to feed job
- Storage Lens: differentiate **free vs advanced metrics**
