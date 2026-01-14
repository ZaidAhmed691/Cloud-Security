# 1. EC2 Storage – Elastic Block Store (EBS) Volumes

## Overview
- **EBS (Elastic Block Store)** is the primary storage option for EC2
- EBS volumes are **network-attached block storage**
- Data on EBS can **persist beyond the life of an EC2 instance**
- Commonly used even when you don’t explicitly notice it (root volumes)

## What is an EBS Volume?
- Network drive attached to an EC2 instance
- Used to **store persistent data**
- Can be detached from one instance and attached to another
- Think of it as a **network-based USB drive**

## Key Characteristics
- **Network-based**
  - Communicates with EC2 over the network
  - Slight latency compared to local disks
- **One instance at a time** (at CCP level)
- **AZ-bound**
  - An EBS volume is created in a specific **Availability Zone**
  - Can only be attached to EC2 instances in the **same AZ**

## Persistence & Failover
- EBS volumes **persist after instance termination** (if configured)
- Can be reattached to a new EC2 instance to recover data
- Very useful for:
  - Data durability
  - Failover scenarios

## Capacity & Performance
- You must **provision capacity in advance**
  - Size (GB)
  - Performance (IOPS)
- You are billed for:
  - Provisioned storage
  - Provisioned performance
- Volumes can be:
  - **Resized**
  - Performance-adjusted over time

## EBS & EC2 Relationships
- One EC2 instance:
  - Can have **multiple EBS volumes** attached
- One EBS volume:
  - Can be attached to **only one EC2 instance at a time**
- EBS volumes:
  - Can exist **unattached**
  - Can be attached on demand

## Availability Zone Constraint
- EC2 instances and EBS volumes are both **AZ-scoped**
- To move data across AZs:
  - Take an **EBS Snapshot**
  - Restore the snapshot in another AZ

## Delete on Termination (Exam-Relevant)
- Controls what happens to EBS volumes when an EC2 instance is terminated
- Defaults:
  - **Root volume** → Delete on termination = **Enabled**
  - **Additional EBS volumes** → Delete on termination = **Disabled**
- Behavior can be customized per volume

## Common Use Case
- Preserve data after instance termination:
  - Disable delete on termination for the root volume
- Exam scenarios often test:
  - Data persistence
  - Root vs additional volume behavior

## Mental Model
- EBS = network disk
- Persistent storage
- AZ-bound
- Detachable & reusable
- Delete-on-termination controls lifecycle

## Exam Takeaways
- EBS volumes persist data
- Bound to a single AZ
- One instance attachment at a time
- Must provision size & IOPS
- Delete on termination is configurable

---

# 2. EBS Volumes – Hands-On Attachment & Lifecycle

## Viewing EBS Volumes on an EC2 Instance
- EC2 → Instance → **Storage** tab
- Shows:
  - Root device
  - Attached block devices
- Default setup:
  - One **root EBS volume** (e.g., 8 GB)
  - Volume status: **in-use**
  - Attached to the instance

## Accessing the Volumes Console
- EC2 → **Volumes**
- Displays:
  - All EBS volumes
  - Size, AZ, state (in-use / available)
  - Attached instance (if any)

## Creating an Additional EBS Volume
- EC2 → Volumes → **Create Volume**
- Choose:
  - Volume type (e.g., `gp2`)
  - Size (e.g., 2 GB)
  - **Availability Zone**
- AZ must match the EC2 instance AZ
  - Example:
    - Instance AZ: `eu-west-1b`
    - Volume AZ must also be `eu-west-1b`

## Attaching an EBS Volume
- Volume state must be **available**
- Actions → **Attach volume**
- Select:
  - Target EC2 instance
- After attachment:
  - Instance now has **multiple block devices**
  - Verified under EC2 → Storage tab

## Using the New Volume
- Newly attached volumes:
  - Are **not formatted**
  - Are **not mounted**
- Requires Linux commands to:
  - Format
  - Mount
- This step is **out of scope** for CCP-level knowledge

## AZ Constraint Demonstration
- Create a volume in a **different AZ** (e.g., `eu-west-1a`)
- Attempt to attach to instance in `eu-west-1b`
- Result:
  - **Attachment fails**
- Confirms:
  - EBS volumes are **AZ-bound**

## Deleting EBS Volumes
- Volumes can be:
  - Created
  - Attached
  - Detached
  - Deleted
- All actions take **seconds**
- Demonstrates elasticity of cloud storage

## Delete on Termination Behavior
- Found under:
  - EC2 → Instance → Storage → Block devices
- Defaults:
  - **Root volume** → Delete on termination = **Yes**
  - **Additional volumes** → Delete on termination = **No**

## Instance Termination Effect
- Terminate EC2 instance:
  - Root EBS volume is **deleted**
  - Additional EBS volumes are:
    - **Detached**
    - **Preserved**
- Verified by:
  - Refreshing Volumes console

## Key Takeaways
- EBS volumes must be in the **same AZ** as the instance
- One instance can have **multiple EBS volumes**
- Root volume is deleted by default on termination
- Additional volumes persist unless explicitly deleted
- Delete-on-termination is **configurable**
- EBS lifecycle is a **common exam scenario**
