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
