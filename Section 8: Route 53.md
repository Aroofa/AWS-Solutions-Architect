## Lesson 1: What is a DNS?
Domain Name System which translates the human friendly hostname into the machine IP Addresses  
www.google.com => 172.2.17.18.36  
DNS is the backbone of the Internet  
DNS uses hierarchical naming structure

- .com  
- example.com  
- www.example.com  
- api.example.com  

### DNS Terminologies
- Domain Registrar: Amazon Route 53, GoDaddy
- DNS Records: A, AAAA, CNAME, NS,..
- Zone File: contains DNS records
- Name Server: resolves DNS queries (Authoritative or Non-Authoritative
- Top Level Domain (TLD): .com, .us, .in, .gov, .org
- Second Level Domain (SLD): amazon.com, google.com

---

## Lesson 2: Route 53 Overview
A highly available, scalable, fully managed and Authoritative DNS  
Authoritative = the customer (you) can update the DNS records  
Route 53 is also a Domain Registrar  
Ability to check the health of your resources  
The only AWS service which provide 100% availability SLA  
Why Route 53? 53 is a reference to the traditional DNS port  

---

## Route 53 - Records
How you want to route traffic for a domain

Each record contains:
- Domain/Subdomain Name - e.g., example.com
- Record Type - e.g., A or AAAA
- Value - e.g., 12.34.56.78
- Routing Policy - how Route 53 responds to queries
- TTL - amount of time the record cached at DNS Resolvers

Route 53 supports the following DNS record types:
- (must know) A / AAAA / CNAME / NS  
- (advanced) CAA / DS / MX / NAPTR / PTR / SOA / TXT / SPF / SRV  

---

## Route 53 - Record Types
- **A** - maps a hostname to IPV4  
- **AAAA** - maps a hostname to IPV6  
- **CNAME** - maps a hostname to another hostname  
  - The target is a domain name which must have an A or AAAA record  
  - Can’t create a CNAME record for the top node of a DNS namespace (Zone Apex)  
  - Example: you can’t create for example.com, but you can create for www.example.com  

- **NS** - Name Servers for the Hosted Zone  
  - Control how traffic is routed for a domain  

---

## Route 53 - Hosted Zones
A container for records that define how to route traffic to a domain and its subdomains

- **Public Hosted Zones** - contains records that specify how to route traffic on the Internet (public domain names)
- **Private Hosted Zones** - contains records that specify how to route traffic within one or more VPCc (private domain names) application1.company.internal

Cost: $0.50 per month per hosted zone  

---

## Lesson 3: Route 53 - Records TTL (Time To Live)

**High TTL** - e.g., 24 hr  
- Less traffic on Route 53  
- Possibly outdated records  

**Low TTL** - e.g., 60 sec.  
- More traffic on Route 53 ($$)  
- Records are outdated for less time  
- Easy to change records  

Except for Alias records, TTL is mandatory for each DNS record

---

## Lesson 4: CNAME vs Alias

AWS Resources (Load Balancer, CloudFront…) expose an AWS hostname  
Ib1-1234.us-east-2.elb.amazonaws.com and you want myapp.mydomain.com  

### CNAME:
- Points a hostname to any other hostname (app.mydomain.com => blabla.anything.com)
- ONLY FOR NON ROOT DOMAIN (aka something.mydomain.com)

### Alias:
- Points a hostname to an AWS Resource (app.mydomain.com => blabla.amazonaws.com)
- Works for ROOT DOMAIN and NON ROOT DOMAIN (aka mydomain.com)
- Free of charge
- Native health check

### Route 53 - Alias Records
- Maps a hostname to an AWS resource  
- An extension to DNS functionality  
- Automatically recognizes changes in the resource’s IP addresses  
- Unlike CNAME, it can be used for the top noe of a DNS namespace (Zone Apex), e.g,; example.com  
- Alias Record is always of type A/AAAA for AWS resources (IPV4 / IPV6)  
- You can’t set the TTL  

### Route 53 - Alias Records Targets
- Elastic Load Balancers  
- CloudFront Distributions  
- API Gateway  
- Elastic Beanstalk environments  
- S3 Websites  
- VPC Interface Endpoints  
- Global Accelerator accelerator  
- Route 53 record in the same hosted zone  

You cannot set an ALIAS record for an EC2 DNS name  

---

## Lesson 5: Routing Policy - Simple

Define how Route 53 responds to DNS queries  
Don’t get confused by the word “Routing”  
It’s not the same as Load balancer routing which routes the traffic  
DNS does not route any traffic, it only responds to the DNS queries

Route 53 Supports the following Routing Policies:
- Simple  
- Weighted  
- Failover  
- Latency based  
- Geolocation  
- Multi-Value Answer  
- Geoproximity (using Route 53 Traffic Flow feature)

### Routing Policies - Simple
- Typically, route traffic to a single resource  
- Can specify multiple values in the same record  
- If multiple values are returned, a random one is chosen by the client  
- When Alias enabled, specify only one AWS resource  
- Can’t be associated with Health Checks  

---

## Lesson 6: Routing Policy - Weighted

- Control the % of the requests that go to each specific resource  
- Assign each record a relative weight:  
  Traffic (%) = Weight for a specific record / sum of all the weights for all records  
- Weights don’t need to sum up to 100  
- DNS records must have the same name and type  
- Can be associated with Health Checks  
- Use cases: load balancing between regions, testing new application versions…  
- Assign a weight of 0 to a record to stop sending traffic to a resource  
- If all records have weight of 0, then all records will be returned equally  

---

## Lesson 7: Routing Policy - Latency
- Redirect to the resource that has the least latency close to us  
- Super helpful when latency for users is a priority  
- Latency is based on traffic between users and AWS Regions  
- Germany users may be directed to the US (if that’s the lowest latency)  
- Can be associated with Health Checks (has a failover capability)  

---

## Lesson 8: Route 53 - Health Checks

HTTP Health Checks are only for public resources  

### Health Check => Automated DNS Failover:
- Health checks that monitor an endpoint (application, server, other AWS resource)
- Health checks that monitor other health checks (Calulated Health Checks)
- Health checks that monitor CloudWatch Alarms (full control!!) - e.g., throttles of DynamoDB, alarms on RDS, custom metrics … (helpful for private resources)

Health checks are integrated with CVV metrics  

### Health Check - Monitor an Endpoint
- About 15 global health checker will check the endpoint health  
- healthy/unhealthy Threshold - 3 (default)  
- Interval - 30 sec (can set to 20 sec - higher cost)  
- Supported protocol: HTTP, HTTPS, and TCP  
- If > 18% of health checkers report the endpoint is healthy, Route 53 considers it Healthy. Otherwise , it’s Unhealthy  
- Ability to show which locations you want Route 53 to use  
- Health checks pass only when the endpoint responds with the 2xx and 3xx status codes  
- Health checks can be setup to pass / fail based on the text in the first 5120 bytes of the response  
- Configure your router/firewall to allow incoming requests from Route 53 Health Checkers  

---

## Route 53 - Calculated Health Checks
- Combine the results multiple health checks into a single health check  
- You can use OR, AND, or NOT  
- Can monitor up to 256 Child Health Checks  
- Specify how many of the health checks need to pass to make the parent pass  
- Usage: perform maintenance to your website without causing all health checks to fail  

---

## Health Checks - Private Hosted Zones
- Route 53 health checkers are outside the VPC  
- They can’t access private endpoints (private VPC or on-premises resource)  
- You can create a CloudWatch Metric and associate a CloudWatch Alarm, then create a Health Check that checks the alarm itself  

---

## Lesson 9: Routing Policy - Failover

---

## Lesson 10: Routing Policy - Geolocation
Different for laterncy-based!  
This routing is based on user location  
Specify location by Continent, Country of by US State (if there’s overlapping, most precise location selected)  
Should create a “Default” record (in case there’s no match on location)  
Use Cases: website localization, restrict content distribution, load balancing, ...  
Can be associated with Health Checks  

---

## Lesson 11: Routing Policy - Geoproximity
Route traffic to your resources based on the geographic location of users and resources  
Ability to shift more traffic to resources based on the defined bias  
To change the size more traffic to resources based on the defined bias  
To expand (1 to 99) - more traffic to the resource  
To shrink (-1 to -99) - less traffic to the resource  

Resources can be:
- AWS resources (specify AWS region)
- Non-AWS resources (specify Latitude and Longitude)

You must use Route 53 Traffic Flow (advanced) to use this feature  

---

## Lesson 12: Routing Policy - IP-based
Routing is based on clients’ IP addresses  
You provide a list of CIDRs for your clients and the corresponding endpoints/locations (user-IP-to-endpoint mappings)  
Use Case: Optimize performance, reduce network costs…  
Example: route end users from a particular ISP to a specific endpoint  

---

## Lesson 13: Routing Policy - Multi Value
Use when routing traffic to multiple resources  
Route 53 return multiple values/resources  
Can be associated with Health Checks (return only values for healthy resources)  
Up to 8 healthy records are returned for each Multi-Value query  
Multi-Value is not a substitute for having an ELB  

---

## Lesson 14: 3rd Party Domains & Route 53
You buy or register your domain name with Domain registrar typically by paying annual charges (e.g., GoDaddy, Amazon Registrar Inc., …)  
The Domain registrar usually provided you with a DNS service to manage your DNS records  
But you can use another DNS service to manage your DNS records  

Example: purchase the domain from GoDaddy and use Route 53 to manage your DNS records  

---

## 3rd Party Registrar with Amazon Route 53
If you buy you domain on a 3rd party registrar, you can still use Route 53 as the DNS Service provider  
Create a Hosted Zone in Route 53  
Update NS Records on 3rd party website to use Route 53 Name Servers  
Domain Registrar != DNS Service  
But every Domain Registrar usually comes with some DNS features  

---

## Lesson 15: Resolvers & Hybrid DNS
Be default, Route 53 Resolver automatically answers DNS queries for:
- Local domain name for EC2 instances
- Records in Private hosted Zones
- Records in public Name Servers

Hybrid DNS - resolving DNS queries between VPC (Route 53 Resolver) and your networks (other DNS Resolvers)

Networks can be:
- VPC itself / Peered VPC
- On-premises Network (connected through Direct Connect or AWS VPN)

### Route 53 - Resolver Endpoints
- Inbound Endpoint - allow your DNS Resolver to resolve domain names for AWS resources (e.g., EC2 instances) and records in Private Hosted Zones
- Outbound Endpoint - Route 53 Resolver forwards DNS queries to your DNS Resolvers

---

# Section 11: Classic Solutions Architecture Discussions
## Lesson 1: Solutions Architectur
