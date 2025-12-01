# Lesson 1: S3 Encryption

---

## 1. Encryption Methods Overview

| Method        | Key Management      | Encryption Location | Header / How to Enable | Notes / Exam Tips |
|---------------|------------------|------------------|----------------------|-----------------|
| **SSE-S3**    | AWS-managed        | Server-side       | `x-amz-server-side-encryption: AES256` | Default for new buckets, AES-256 |
| **SSE-KMS**   | AWS KMS-managed    | Server-side       | `x-amz-server-side-encryption: aws:kms` | Audit logs via CloudTrail, API limits apply (5k–30k/sec) |
| **SSE-C**     | Customer-provided  | Server-side       | Provide key in HTTP headers | Key never stored, HTTPS required |
| **Client-Side** | Customer           | Client-side       | Encrypt locally before upload | Full control over keys & encryption cycle, use Client-Side Encryption Library |

---

## 2. Encryption in Transit (Data in Flight)

- **Also called:** SSL / TLS
- **Endpoints:**
  - HTTP → Not encrypted
  - HTTPS → Encrypted
- **Recommendation:** Always use HTTPS
- **SSE-C:** HTTPS is mandatory for key transmission

**Enforcing HTTPS with Bucket Policy:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::your-bucket-name/*",
      "Condition": {
        "Bool": {"aws:SecureTransport": "false"}
      }
    }
  ]
}
```

## 3. Encryption Flow Diagram
```mermaid
flowchart TD
    A[Client Upload] --> B[Choose Encryption Method]
    B --> C[SSE-S3: AWS-managed key → Encrypted object in S3]
    B --> D[SSE-KMS: KMS key → Encrypted object in S3 (requires KMS access to read)]
    B --> E[SSE-C: Customer key → Encrypted object in S3 (key never stored, HTTPS required)]
    B --> F[Client-Side: Encrypt locally → Upload → Decrypt after download]

````

## 4. Quick Exam Tips
- **SSE-S3**: Easy to use, default encryption.
- **SSE-KMS**: Strong control, auditing, but be aware of API request limits.
- **SSE-C**: Customer manages keys; HTTPS required.
- **Client-side encryption**: Complete control outside AWS.

> Always enforce HTTPS to secure data in transit.
