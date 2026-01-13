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

# 5. IAM Roles for EC2 – Hands-On Usage

## Overview
- EC2 instances often need to call AWS APIs (via AWS CLI)
- **Correct and secure way** to provide credentials is using **IAM Roles**
- **Never** store or configure IAM access keys directly on an EC2 instance

## Connecting to the EC2 Instance
- Access can be done via:
  - SSH
  - EC2 Instance Connect (browser-based)
- Once connected:
  - You are in a standard **Linux terminal**
  - Commands behave the same regardless of connection method

## AWS CLI on EC2
- Amazon Linux AMI comes with **AWS CLI pre-installed**
- You can immediately run AWS CLI commands
- Example:
  - `aws iam list-users`
- Without credentials, AWS returns:
  - **Unable to locate credentials**

## Why NOT `aws configure` on EC2
- `aws configure` stores:
  - Access Key ID
  - Secret Access Key
- This is **extremely insecure**
- Anyone with access to the instance can retrieve the credentials
- **Rule of thumb**:
  - Never enter IAM access keys on EC2 instances

## Using IAM Roles (Correct Approach)
- IAM Roles provide **temporary credentials** automatically
- Credentials are managed securely by AWS
- No access keys are stored on the instance

## Attaching an IAM Role to EC2
- Go to:
  - EC2 → Instances → Actions → Security → Modify IAM role
- Select an existing IAM role
- Save changes
- Role is attached **immediately** to the instance

## Verifying Role Access
- After attaching the role:
  - Run AWS CLI commands again
  - Example:
    - `aws iam list-users`
- Command works **without running `aws configure`**

## Permission Enforcement
- If permissions are removed from the role:
  - AWS CLI returns **AccessDenied**
- IAM permissions are enforced **in real time**
- Policy changes may take a few seconds to propagate

## Key Behavior to Remember
- IAM role permissions:
  - Are directly tied to the EC2 instance
  - Change behavior of AWS CLI commands instantly
- No credentials are visible or manually managed

## Best Practices
- Always use **IAM roles** for EC2
- Assign **least-privilege policies**
- Never hardcode or configure access keys on instances

## Exam Takeaways
- EC2 credentials should be provided via **IAM Roles only**
- `aws configure` on EC2 = **security anti-pattern**
- IAM roles use **temporary credentials**
- Permissions can be modified without restarting the instance

---

# 6. EC2 Instance Purchasing Options

## Overview
- AWS offers multiple **EC2 purchasing options** to optimize **cost vs flexibility**
- Choice depends on:
  - Workload duration
  - Predictability
  - Fault tolerance
  - Compliance or licensing needs
- Exam focuses on **matching the right option to the workload**

## On-Demand Instances
- Pay for what you use
- Billing:
  - Per second (Linux / Windows, after first minute)
  - Per hour (other OS)
- No upfront payment
- No long-term commitment
- Highest cost
- Best for:
  - Short-term workloads
  - Unpredictable usage
  - Applications that cannot be interrupted

## Reserved Instances (RI)
- Up to ~72% discount vs On-Demand
- Commitment:
  - 1 year or 3 years
- Reserve specific attributes:
  - Instance type
  - Region
  - OS
  - Tenancy
- Payment options:
  - No upfront
  - Partial upfront
  - All upfront (maximum discount)
- Scope:
  - Regional or Zonal (AZ capacity reservation)
- Best for:
  - Steady-state workloads
  - Long-running databases
- Can be **bought and sold** on the Reserved Instance Marketplace

## Convertible Reserved Instances
- More flexibility than standard RIs
- Can change:
  - Instance family
  - Instance type
  - OS
  - Tenancy
  - Scope
- Lower discount than standard RIs (up to ~66%)
- Best for:
  - Long-term workloads with changing requirements

## Savings Plans
- Modern alternative to Reserved Instances
- Discount up to ~70% (similar to RIs)
- Commitment:
  - Spend a fixed **$ per hour** for 1 or 3 years
- Flexibility:
  - Instance size
  - OS
  - Tenancy
- Locked to:
  - Instance family
  - Region
- Usage beyond commitment billed at On-Demand rates
- Best for:
  - Long-term workloads with variable instance sizes

## Spot Instances
- Up to ~90% discount vs On-Demand
- Instances can be **terminated at any time**
- You set a maximum price
- Best for:
  - Batch jobs
  - Data analysis
  - Image processing
  - Distributed workloads
  - Fault-tolerant jobs
- NOT suitable for:
  - Databases
  - Critical workloads
- Highly tested in exams

## Dedicated Hosts
- Entire **physical server** dedicated to you
- Full control over:
  - Instance placement
  - Underlying hardware
- Supports **Bring Your Own License (BYOL)**
- Pricing:
  - On-Demand (per second)
  - Reserved (1 or 3 years)
- Most expensive option
- Best for:
  - Compliance requirements
  - Server-bound software licenses
  - Regulatory constraints

## Dedicated Instances
- Instances run on **dedicated hardware**
- Hardware may be shared with other instances **in your account**
- No control over instance placement
- Less visibility than Dedicated Hosts
- Difference from Dedicated Hosts:
  - Dedicated Host → physical server access
  - Dedicated Instance → isolated instances only

## Capacity Reservations
- Reserve **On-Demand capacity** in a specific AZ
- No time commitment
- No billing discounts
- Charged whether instances run or not
- Guarantees capacity availability
- Can be combined with:
  - Regional Reserved Instances
  - Savings Plans (for cost optimization)
- Best for:
  - Short-term, uninterrupted workloads
  - AZ-specific capacity requirements

## Mental Model (Exam-Friendly)
- On-Demand → Pay full price, maximum flexibility
- Reserved → Commit long-term, maximum discount
- Savings Plan → Commit to spend, flexible usage
- Spot → Cheapest, interruptible
- Dedicated Host → Full physical server control
- Capacity Reservation → Guarantee capacity, no discount

## Exam Takeaways
- Focus on **workload characteristics**
- Discounts vs flexibility is the core trade-off
- Spot ≠ critical workloads
- Reserved / Savings Plans = long-term usage
- Capacity Reservation = availability, not savings

---

# 7. EC2 Spot Instances – Deep Dive

## Overview
- **Spot Instances** provide up to **90% discount** vs On-Demand
- Ideal for **fault-tolerant, non-critical workloads**
- AWS can reclaim instances based on **spot price changes**

## How Spot Pricing Works
- You define a **maximum spot price** you’re willing to pay
- AWS spot price:
  - Varies by **supply and demand**
  - Varies by **Availability Zone**
- Instance runs as long as:
  - Current spot price ≤ your max price
- If spot price exceeds your max price:
  - Instance may be reclaimed by AWS

## Interruption Behavior
- AWS provides a **2-minute warning (grace period)**
- During this time, you can:
  - **Stop** the instance  
    - Preserve state (EBS-backed)
    - Restart when price drops again
  - **Terminate** the instance  
    - Start fresh later
- Choice depends on whether workload state matters

## Spot Blocks
- Reserve a Spot Instance for **1 to 6 hours**
- Designed to prevent interruptions during the block window
- Rare interruptions may still occur
- Useful for:
  - Time-bound batch jobs
  - Predictable short workloads

## When to Use Spot Instances
- Best for:
  - Batch processing
  - Data analysis
  - Image / video processing
  - Distributed workloads
  - Flexible start/end jobs
- NOT suitable for:
  - Databases
  - Critical workloads
  - Stateful, non-resilient applications

## Spot Price Characteristics
- Spot prices:
  - Differ per **Availability Zone**
  - Can fluctuate over time
- Often relatively stable
- Still significantly cheaper than On-Demand
- If your max price is set **well above** current prices:
  - Instance is unlikely to be reclaimed

## Spot Requests
- A Spot Request defines:
  - Number of instances
  - Maximum price
  - Launch specification (AMI, instance type, etc.)
  - Valid-from / valid-until time
  - Request type

## Spot Request Types
- **One-Time**
  - Instance launches once
  - Request is removed after fulfillment
- **Persistent**
  - AWS maintains desired capacity
  - If instance is interrupted:
    - AWS attempts to relaunch automatically
  - Request remains active until canceled

## Correct Way to Terminate Spot Instances
- To permanently stop Spot usage:
  1. **Cancel the Spot Request**
  2. **Terminate the Spot Instances**
- If you terminate instances first:
  - Persistent request will relaunch them automatically
- This order is **exam-relevant**

## Spot Fleets
- A **Spot Fleet** manages:
  - Multiple Spot Instances
  - Optionally On-Demand Instances
- Fleet tries to meet:
  - Target capacity
  - Budget constraints
- Uses **multiple launch pools**:
  - Different instance types
  - Different AZs
  - Different OS options

## Spot Fleet Allocation Strategies
- **Lowest Price**
  - Launches from cheapest pool
  - Best for short workloads
  - Most exam-tested
- **Diversified**
  - Spreads instances across pools
  - Improves availability
  - Good for long workloads
- **Capacity Optimized**
  - Chooses pool with highest available capacity
- **Price-Capacity Optimized**
  - Chooses highest-capacity pool
  - Then selects lowest price
  - Best general-purpose strategy

## Why Use Spot Fleets
- More savings than single Spot Instance requests
- Automatic optimization across:
  - Instance types
  - Availability Zones
- Ideal when:
  - You need raw compute power
  - You don’t care about exact instance type

## Exam Takeaways
- Spot = **cheap but interruptible**
- Always associate Spot with **fault tolerance**
- Persistent requests auto-relaunch instances
- Cancel Spot Requests **before** terminating instances
- Spot Fleets = **maximum flexibility + maximum savings**
- Lowest-price strategy is the most common exam answer

---
