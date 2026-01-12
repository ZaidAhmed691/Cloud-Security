# 1. EC2 Instance Types

## Overview
- EC2 instances come in **different types** optimized for **different workloads**
- Categories are based on **compute, memory, storage, and networking**
- The AWS EC2 Instance Types page is the **source of truth** for specs and pricing
- Instance families evolve over time

## EC2 Naming Convention
Example: `m5.2xlarge`

- `m` → Instance class (workload optimization)
- `5` → Instance generation (newer hardware)
- `2xlarge` → Instance size  
  → Larger size = more vCPU, more RAM, higher performance

## General Purpose Instances
- Balanced **CPU, memory, and networking**
- Suitable for **diverse workloads**
- Common use cases:
  - Web servers
  - Application servers
  - Code repositories
- Example:
  - `t2.micro` (Free Tier, used in labs)

## Compute Optimized Instances
- Designed for **CPU-intensive workloads**
- Prioritize **high compute performance**
- Common use cases:
  - Batch processing
  - Media transcoding
  - High-performance web servers
  - High Performance Computing (HPC)
  - Machine learning
  - Gaming servers
- Instance family:
  - `C` series (e.g., `c5`, `c6`)

## Memory Optimized Instances
- Designed for **large datasets in RAM**
- Emphasis on **fast memory access**
- Common use cases:
  - In-memory databases
  - Relational and NoSQL databases
  - Distributed caching (e.g., ElastiCache)
  - Business Intelligence (BI)
  - Real-time big data processing
- Instance families:
  - `R` series
  - `X1`, High Memory, `Z1`

## Storage Optimized Instances
- Designed for **high disk I/O and local storage**
- Best for **data-intensive workloads**
- Common use cases:
  - OLTP systems
  - Relational and NoSQL databases
  - In-memory database cache (e.g., Redis)
  - Data warehousing
  - Distributed file systems
- Instance families:
  - `I`, `D`, `H1`

## Conceptual Instance Comparison
- `t2.micro`
  - 1 vCPU
  - 1 GB RAM

- `r5.16xlarge`
  - Memory-heavy
  - Very large RAM footprint

- `c5.d.4xlarge`
  - High CPU
  - Lower memory relative to RAM-optimized instances

→ Instance families prioritize **different resource dimensions**

## Useful Tools
- **ec2instances.info**
  - Compare all EC2 instances
  - View vCPU, memory, pricing, and bandwidth
  - Filter and sort instances easily

## Exam Takeaways
- Focus on **workload → instance type mapping**
- Exact specs do **not** need memorization
- Key associations:
  - General Purpose → balanced workloads
  - Compute Optimized → CPU-heavy workloads
  - Memory Optimized → RAM-heavy workloads
  - Storage Optimized → high I/O workloads
- EC2 naming conventions are **conceptually tested**

---

# 2. EC2 Security Groups (Firewalls)

## Overview
- **Security Groups** act as a **firewall around EC2 instances**
- Control **inbound (incoming)** and **outbound (outgoing)** traffic
- Fundamental building block for **network security in AWS**
- Rules define **what is allowed**, never what is denied

## Key Characteristics
- Only **ALLOW rules** (no explicit deny rules)
- Rules can reference:
  - **IP address ranges** (IPv4 / IPv6)
  - **Other security groups**
- Evaluated **outside the EC2 instance**
  - Blocked traffic never reaches the instance

## Traffic Direction
- **Inbound rules**
  - Control traffic from outside → EC2 instance
- **Outbound rules**
  - Control traffic from EC2 instance → outside
- Defaults:
  - All **inbound traffic blocked**
  - All **outbound traffic allowed**

## Rule Structure
- Rule components:
  - Type (e.g., SSH, HTTP)
  - Protocol (TCP, etc.)
  - Port number
  - Source (IP range or security group)
- `0.0.0.0/0` means **allow traffic from anywhere**

## Example Behavior
- If your IP is allowed on port 22 → SSH access works
- If another IP is not allowed → connection **times out**
- Timeout = **security group issue**
- Connection refused = **application issue** (traffic reached instance)

## Security Group Associations
- One security group can be attached to **multiple EC2 instances**
- One EC2 instance can have **multiple security groups**
- Security groups are scoped to a **Region + VPC**
  - Cannot be reused across regions or VPCs

## Best Practices
- Keep a **separate security group for SSH access**
- Makes troubleshooting and access control simpler
- SSH is usually the most sensitive access point

## Referencing Security Groups
- Security groups can allow traffic **from other security groups**
- Enables instance-to-instance communication **without using IPs**
- Common use case:
  - Load balancers
  - Multi-tier architectures
- Instances with authorized security groups can communicate automatically
- Instances with unauthorized security groups are blocked

## Common Ports to Know (Exam-Relevant)
- **22** → SSH (Linux login)
- **21** → FTP (File Transfer Protocol)
- **22** → SFTP (Secure File Transfer over SSH)
- **80** → HTTP (Unsecured web traffic)
- **443** → HTTPS (Secured web traffic)
- **3389** → RDP (Windows login)

## Exam Takeaways
- Security groups are **stateful firewalls**
- Only allow rules exist
- Timeout errors usually indicate **security group misconfiguration**
- Port numbers and their purposes are **highly testable**

---

