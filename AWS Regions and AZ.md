# AWS Cloud Overview - Regions and AZ

## AWS Cloud Use Cases
AWS enables you to build sophisticated, scalable applications  
Applicable to a diverse set of industries  

### Use Cases include
- Enterprise IT, Backup and Storage, Big Data analytics  
- Website hosting Mobile and Social Apps  
- Gaming  

---

## AWS Global Infrastructure

### AWS Regions
AWS has regions all around the world  
Names can be us-east-1, eu-west-3…  
A region is a cluster of data centers  
Most AWS services are region-scoped  

#### How to choose an AWS Region?
- Compliance with data governance and legal requirements: data never leaves a region without your explicit permission  
- Proximity to customers: reduced latency  
- Available services within a Region: new services and new feature aren’t available in every region  
- Pricing: pricing varies region to region and is transparent in the service pricing page  

### AWS Availability Zones
Each region has many availability zones (usually 3, min is 3, max is 6)  
Example: ap-southeast-2a, ap-southeast-2b, ap-southeast-2c  
Each availability zone (AZ) is one or more discrete data centers with redundant power, networking, and connectivity  
They’re separate from each other, so that they’re isolated from disasters  
They’re connected with high bandwidth, ultra-low latency networking  

### AWS Data Centers

### AWS Edge Locations / Points of Presence
Amazon has 400+ Point of Presence (400+ Edge Locations and 10+ Regional Caches) in 90+ cities across 40+ countries  
Content is delivered to end users with lower latency  

---

## Tour of the AWS Console

### AWS has Global Services:
- Identity and Access Management (IAM)  
- Route 53 (DNS service)  
- CloudFront (Content Delivery Network)  
- WAF (WEB Application Firewall)  

### Most AWS services are Region-scoped:
- Amazon EC2 (Infrastructure as a Service)  
- Elastic Beanstalk (Platform as a Service)  
- Lambda (Function as a Service)  
- Rekognition (Software as a Service)  

---

## Region Table
