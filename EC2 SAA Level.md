# 1. EC2 IP Addressing Behavior (Public, Private & Elastic IPs)

## Public vs Private IPv4
- **Public IPv4**
  - Used to access EC2 from the **internet**
  - Required for SSH from your local machine
- **Private IPv4**
  - Used for **internal AWS networking**
  - Works only **inside the VPC**
  - Cannot be used to SSH from the public internet

## Connectivity Behavior
- SSH using **public IPv4** → works
- SSH using **private IPv4** from your laptop → fails
- Reason:
  - Your local machine is not inside AWS’s private network
  - Private IPs are not routable over the internet

## Instance Stop vs Reboot
- **Reboot**
  - Public IP stays the same
- **Stop + Start**
  - Public IPv4 **changes**
  - Private IPv4 **remains the same**
- Important when relying on static IP access

## Problem: Public IP Changes
- Stopping and starting an instance causes:
  - Loss of the previous public IPv4
- Breaks:
  - SSH access
  - Hardcoded IP references
  - External integrations

## Solution: Elastic IP (EIP)
- **Elastic IP**
  - Static public IPv4 address
  - Allocated from AWS and **owned by you while allocated**
  - Can be attached to:
    - EC2 instance
    - Network Interface (ENI)
- Public IPv4 stays the same across:
  - Stop / Start
  - Reboots

## Elastic IP Pricing (Important)
- Public IPv4s (including EIP):
  - Charged **~$0.005 per hour**
  - Charged whether used or unused
- Free tier:
  - 750 hours/month of public IPv4
- Best practice:
  - **Release Elastic IPs when not in use**
  - Terminate unused EC2 instances

## Associating an Elastic IP
- Allocate Elastic IP from:
  - EC2 → Elastic IPs
- Associate with:
  - EC2 instance
  - Specific private IP
- After association:
  - Instance public IPv4 = Elastic IP

## Behavior with Elastic IP
- Stop instance:
  - Elastic IP remains attached
- Start instance:
  - Same public IPv4 is reused
- SSH continues to work using the same IP

## Cleanup Best Practices
- Disassociate Elastic IP when no longer needed
- Release Elastic IP to avoid charges
- Terminate EC2 instances after labs

## Exam Takeaways
- Public IP changes on stop/start
- Private IP never changes
- Elastic IP = static public IPv4
- Elastic IPs cost money if left allocated
- Use Elastic IP when a **fixed public IP is required**

---

# 2. EC2 Placement Groups

## Overview
- **Placement Groups** control how EC2 instances are placed on AWS hardware
- Used when you need:
  - High performance networking
  - Fault isolation
  - Predictable failure domains
- You don’t control hardware directly, but you **influence placement strategy**

## Placement Group Strategies
- AWS provides **three placement group types**:
  - Cluster
  - Spread
  - Partition

---

## Cluster Placement Group
- Instances are placed **close together**
- Located within **one Availability Zone**
- Optimized for:
  - **Low latency**
  - **High throughput networking**
- Supports enhanced networking (up to ~10 Gbps)

### Pros
- Extremely fast inter-instance communication
- Ideal for tightly coupled workloads

### Cons
- **High risk**
- If the AZ fails, **all instances fail together**

### Use Cases
- Big data jobs that must finish quickly
- High-performance computing (HPC)
- Applications requiring very low latency between instances

---

## Spread Placement Group
- Instances are placed on **distinct hardware**
- Can span **multiple Availability Zones**
- Designed to **minimize correlated failures**

### Key Characteristics
- Maximum **7 instances per AZ per placement group**
- Each instance runs on separate hardware

### Pros
- Strong isolation
- Reduced risk of simultaneous instance failure

### Cons
- Hard limit on scale (small groups only)

### Use Cases
- Critical applications
- Systems requiring maximum availability
- Workloads where instance failures must be isolated

---

## Partition Placement Group
- Instances are grouped into **partitions**
- Each partition uses **separate hardware racks**
- Partitions can span **multiple AZs**

### Key Characteristics
- Up to **7 partitions per AZ**
- Each partition can contain **many EC2 instances**
- Supports **hundreds of instances** per placement group
- Failure of one partition does not affect others

### Pros
- Scales much larger than Spread
- Strong fault isolation at rack level

### Cons
- Application must be **partition-aware**

### Use Cases
- Large distributed systems
- Big data and streaming platforms:
  - Hadoop
  - HDFS / HBase
  - Cassandra
  - Apache Kafka

---

## Partition Awareness
- Instances can discover their partition using:
  - **EC2 Metadata Service**
- Applications use this info to:
  - Distribute data
  - Handle failures intelligently

---

## Exam Takeaways
- Cluster → **performance**, low latency, high risk
- Spread → **high availability**, small scale
- Partition → **large scale + fault isolation**
- Remember:
  - Spread = max **7 instances per AZ**
  - Partition = max **7 partitions per AZ**
- Partition placement groups are common for **big data workloads**

---

# 3. Placement Groups – Hands-On Practice

## Creating Placement Groups
- Navigate to:
  - EC2 → **Network & Security → Placement Groups**
- Placement groups must be created **before** launching instances

## Cluster Placement Group (High Performance)
- Name: `my-high-performance-group`
- Strategy: **Cluster**
- Purpose:
  - Place instances close together
  - Maximize network throughput and minimize latency
- Use case:
  - High-performance / tightly coupled workloads

## Spread Placement Group (Critical Applications)
- Name: `my-critical-group`
- Strategy: **Spread**
- Spread level:
  - **Rack** (default and exam-relevant)
  - Host (Outposts only, not used here)
- Purpose:
  - Distribute instances across distinct hardware
  - Reduce correlated failures
- Use case:
  - Critical, highly available applications

## Partition Placement Group (Distributed Systems)
- Name: `my-distributed-group`
- Strategy: **Partition**
- Number of partitions:
  - Configurable (e.g., 1–7 per AZ)
- Purpose:
  - Spread instances across multiple hardware racks
  - Enable large-scale fault isolation
- Use case:
  - Big data and distributed systems

## Launching EC2 Instances into a Placement Group
- Click **Launch Instances**
- Scroll to **Advanced Details**
- Select the desired **Placement Group name**
  - Cluster
  - Spread
  - Partition
- Instance will be launched according to the selected strategy

## Key Takeaways
- Placement group is selected **at launch time**
- Different placement strategies serve different goals:
  - Cluster → performance
  - Spread → availability
  - Partition → scalability + fault isolation
- Placement groups are easy to configure but powerful in impact

---

# 4. Elastic Network Interfaces (ENI)

## Overview
- **ENI (Elastic Network Interface)** = virtual network card in a VPC
- Provides **network connectivity** to EC2 instances
- Not exclusive to EC2 (used by other AWS services as well)
- Exists as a **separate logical component** that can be attached/detached

## Core Concepts
- Each EC2 instance has:
  - **Primary ENI** (`eth0`) by default
- ENIs are:
  - Created **inside a VPC**
  - **Bound to a specific Availability Zone**

## ENI Attributes
- Networking:
  - One **primary private IPv4**
  - One or more **secondary private IPv4s**
- Public addressing:
  - One Elastic IP per private IPv4 (optional)
  - Or public IPv4 addresses
- Security:
  - One or more **security groups**
- Identity:
  - MAC address
- Other metadata (not exam-critical)

## Multiple ENIs per EC2
- EC2 instances can have:
  - Multiple ENIs (e.g., `eth0`, `eth1`)
- Each ENI provides:
  - Its own private IP(s)
  - Its own security groups
- Useful for:
  - Multi-homed networking
  - Traffic separation
  - Advanced architectures

## ENI Lifecycle & Flexibility
- ENIs can be:
  - Created **independently** of EC2
  - Attached **on the fly**
  - Detached and reattached to another EC2 instance
- ENIs can be **moved between EC2 instances**
  - As long as they are in the **same AZ**

## ENIs and Failover
- ENIs enable **fast failover** scenarios
- Example:
  - Application accessed via a **private static IP**
  - That IP belongs to an ENI
  - ENI can be detached from Instance A
  - ENI attached to Instance B
- Result:
  - Traffic switches to new instance without IP change

## Availability Zone Constraint
- ENIs are **AZ-scoped**
- An ENI created in AZ-A:
  - Can only be attached to instances in AZ-A
- Cannot move ENIs across AZs

## Use Cases
- Failover using static private IPs
- Multiple security groups per instance
- Advanced networking setups
- Decoupling network identity from compute

## Exam Takeaways
- ENI = virtual network card
- Primary ENI = `eth0`
- Supports multiple private IPs
- Can be detached and reattached
- Used for **failover**
- **AZ-bound** (cannot cross AZs)

---

# 5. Elastic Network Interfaces (ENI) – Hands-On Practice

## Launching Instances
- Launch **two EC2 instances** (Amazon Linux 2, `t2.micro`)
- Use an existing security group (e.g., Launch Wizard)
- After launch:
  - Each instance automatically gets **one ENI**
  - ENI includes:
    - Public IPv4
    - Private IPv4
    - Private DNS
  - ENIs are visible:
    - EC2 Instance → Networking
    - EC2 → Network & Security → Network Interfaces

## Viewing Default ENIs
- Each instance has:
  - One ENI in **in-use** state
  - ENI is linked to a specific **instance ID**
- Default ENIs are created **with the instance**

## Creating a Custom ENI
- Navigate to:
  - EC2 → Network & Security → Network Interfaces → Create
- Configure:
  - Description (e.g., `demo-eni`)
  - Subnet (defines the **AZ**)
  - Private IPv4 (auto-assign or custom)
  - Security group
- Result:
  - New ENI in **available** state
  - Not attached to any instance yet

## Attaching a Custom ENI
- Select ENI → **Attach**
- Choose an EC2 instance in the **same AZ**
- After attachment:
  - Instance now has **two ENIs**
    - Primary ENI (`eth0`) → public + private IP
    - Secondary ENI (`eth1`) → private IP only

## Why Use Multiple ENIs
- Gain control over **private IP ownership**
- Separate network traffic
- Enable **fast failover**
- Advanced networking scenarios

## ENI-Based Failover (Demo)
- Detach the custom ENI from Instance A
- Attach the same ENI to Instance B
- Result:
  - The **private IP moves** with the ENI
  - Traffic using that private IP now reaches Instance B
- Useful when:
  - Both instances run the same application
  - Access is via a static private IP

## Detaching ENIs
- ENI can be detached:
  - Normally
  - Or using **force detach** if needed
- ENI must be detached before attaching to another instance

## ENI Lifecycle on Instance Termination
- Default ENIs (created with instances):
  - **Deleted automatically** when instance is terminated
- Manually created ENIs:
  - **Remain** after instance termination
  - Must be deleted manually if no longer needed

## Cost Considerations
- ENIs themselves:
  - **No additional cost**
- Charges depend on:
  - Associated resources (Elastic IPs, data transfer)

## Key Takeaways
- ENIs can be created independently of EC2
- ENIs are **AZ-bound**
- ENIs can be attached, detached, and moved
- Moving an ENI moves its **private IP**
- Manually created ENIs persist after instance termination
- Powerful but **advanced** networking feature

---

# 14. EC2 Hibernate

## Overview
- **EC2 Hibernate** preserves the **in-memory (RAM) state** of an instance
- Unlike stop/start:
  - The operating system is **frozen**, not rebooted
  - Instance resumes **much faster**
- Ideal when startup time or in-memory state matters

## Instance States Comparison
- **Stop**
  - EBS data is preserved
  - RAM is lost
  - OS boots again on start
- **Terminate**
  - Instance is destroyed
  - Root volume deleted if configured
- **Hibernate**
  - EBS data preserved
  - **RAM state preserved**
  - OS does NOT reboot

## How Hibernate Works
- Instance is running with data in RAM
- On hibernate:
  - RAM contents are written to the **root EBS volume**
  - Instance enters stopping state
- On start:
  - RAM is restored from EBS
  - Instance resumes exactly where it left off

## Requirements for Hibernate
- Root volume must be:
  - **EBS-backed**
  - **Encrypted**
  - Large enough to store the RAM dump
- Instance RAM size must be within supported limits
- Not supported for:
  - Bare metal instances

## Supported Configurations
- Works with:
  - Linux and Windows
  - On-Demand instances
  - Reserved Instances
  - Spot Instances
- Hibernate duration:
  - Intended for **short to medium term** (up to ~60 days)

## Use Cases
- Applications with:
  - Long initialization times
  - Large in-memory caches
  - Long-running processes
- Need fast resume without reloading state
- Avoid repeated OS boot and app startup overhead

## Key Benefits
- Faster startup than stop/start
- Preserves application and OS state
- Reduces initialization time and complexity

## Exam Takeaways
- Hibernate preserves **RAM + EBS**
- Stop preserves **EBS only**
- Root volume must be **encrypted**
- Faster resume than stop/start
- Not supported on bare metal instances
