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

# 3. EC2 Security Groups – Hands-On Behavior & Troubleshooting

## Viewing Security Groups
- Security groups attached to an EC2 instance can be viewed from:
  - EC2 instance → **Security tab**
  - **Networking & Security → Security Groups** (full view)
- Each security group has a **unique ID**
- Common default groups:
  - **Default security group**
  - **Launch Wizard security group** (created during instance launch)

## Inbound Rules (Incoming Traffic)
- Inbound rules control traffic **from outside → EC2**
- Example rules:
  - SSH → Port 22 → Source: `0.0.0.0/0` (anywhere)
  - HTTP → Port 80 → Source: `0.0.0.0/0`
- Port 80 inbound rule allows access to a web server via the instance’s **public IPv4 address**

## Effect of Removing Inbound Rules
- Removing HTTP (port 80) rule:
  - Website becomes inaccessible
  - Browser keeps loading → **timeout**
- Adding the rule back restores access immediately

## Timeout vs Connection Refused (Very Important)
- **Timeout**
  - 100% indicates a **security group issue**
  - Traffic is blocked before reaching the instance
- **Connection refused**
  - Security group allowed traffic
  - Application is not running or misconfigured

## Adding Inbound Rules
- You can allow:
  - Specific ports (e.g., 80, 443)
  - Port ranges
- Rule configuration options:
  - Select protocol from dropdown (e.g., HTTP, HTTPS)
  - Allow from:
    - Anywhere (`0.0.0.0/0`)
    - Your IP only
    - CIDR blocks
    - Other security groups
- Using **My IP** is safer but risky if your IP changes → results in timeout

## Outbound Rules (Outgoing Traffic)
- Outbound rules control traffic **from EC2 → outside**
- Default behavior:
  - Allow **all IPv4 traffic to anywhere**
- This enables full internet access from the EC2 instance

## Multiple Security Groups
- An EC2 instance can have **multiple security groups attached**
- A security group can be attached to **multiple EC2 instances**
- Rules from multiple security groups are **combined (additive)**

## Key Takeaways
- Security groups are the **first place to check** when connectivity fails
- Timeouts almost always mean **incorrect inbound rules**
- Inbound rules enable access; outbound rules enable connectivity from the instance
- Security groups are **reusable and flexible**
- Changes take effect **immediately**

---

# 4. EC2 Instance Connect (Browser-Based SSH)

## Overview
- **EC2 Instance Connect** is an **alternative to traditional SSH**
- Provides **browser-based SSH access** to EC2 instances
- Eliminates the need to manually manage SSH key pairs
- Commonly used for **quick access and demos**

## How EC2 Instance Connect Works
- Accessed via:
  - EC2 Console → Instance → **Connect**
  - Select **EC2 Instance Connect**
- AWS automatically:
  - Uploads a **temporary SSH key**
  - Establishes an SSH session behind the scenes
- Uses the default Linux username:
  - `ec2-user` (for Amazon Linux)
- Session opens in a **new browser tab**

## Key Benefits
- No local terminal required
- No SSH key management
- Works across Windows, macOS, and Linux
- Fast and convenient for learning and troubleshooting

## Behind the Scenes
- EC2 Instance Connect still relies on **SSH**
- Requires:
  - **Port 22 open** in the security group
  - Correct inbound rules for IPv4 and sometimes IPv6

## Security Group Requirements
- Inbound rule required:
  - SSH → Port 22
  - Source:
    - Anywhere IPv4 (`0.0.0.0/0`)
    - Sometimes Anywhere IPv6 (`::/0`)
- If port 22 is closed:
  - Connection fails immediately

## Troubleshooting Connection Issues
- Connection failure usually means:
  - SSH (port 22) is blocked in the security group
- Fix:
  - Add SSH inbound rule
  - Ensure correct IPv4 / IPv6 configuration

## Usage Notes
- Supports running standard Linux commands:
  - `whoami`
  - `ping google.com`
- Can be used instead of:
  - Local SSH terminal
  - PuTTY
  - OS-specific SSH clients

## Course Context
- EC2 Instance Connect will be used frequently
- Whenever SSH is mentioned:
  - You can use either traditional SSH or EC2 Instance Connect

## Exam Takeaways
- EC2 Instance Connect = **SSH via AWS Console**
- Still requires **port 22**
- Temporary SSH keys are managed by AWS
- Simplifies access but does not replace SSH concepts

---
