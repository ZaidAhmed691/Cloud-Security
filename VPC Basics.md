
# 1. AWS VPC & Subnets

> **Goal:** Create a private network in AWS and divide it into subnets for workloads.

---

## What is a VPC?

* A **Virtual Private Cloud (VPC)** is your own **private network inside AWS**
* Logical isolation for EC2, databases, load balancers, etc.
* Fully isolated **by default**

---

## Default VPC

* Created **automatically** by AWS
* Uses CIDR like `172.31.0.0/16`
* Includes:

  * Subnets in each Availability Zone
  * Internet Gateway
  * Route table with internet access

**Exam Tip:**
Default VPC = convenient, **not secure by design**

---

## Custom VPC (Recommended)

### Steps to Create a VPC

1. AWS Console → **Networking & Content Delivery → VPC**
2. Click **Create VPC**
3. Enter **Name tag** (e.g., `SALES`)
4. Enter **IPv4 CIDR** (e.g., `10.10.0.0/16`)
5. Leave IPv6 disabled (optional)
6. Set **Tenancy = Default**
7. Click **Create**

---

## CIDR Basics (High Yield)

* IPv4 address = **32 bits**
* `/16` → large network (~65,536 IPs)
* `/24` → smaller network (~256 IPs)

### Private IP Ranges Allowed in VPCs

* `10.0.0.0/8`
* `172.16.0.0/12`
* `192.168.0.0/16`

**Exam Tip:**
Public IP ranges cannot be used as VPC CIDR blocks

---

## VPC Isolation Rule (Very Important)

* **VPCs cannot communicate with each other by default**
* Applies even within the same AWS account

### To Connect VPCs

* VPC Peering
* VPN
* AWS Direct Connect

**Exam Phrase:**
“VPCs are isolated private networks”

---

## Subnets

### What is a Subnet?

* A **subdivision of a VPC**
* Exists in **one Availability Zone only**

---

### Steps to Create a Subnet

1. VPC Console → **Subnets → Create subnet**
2. Select the **VPC**
3. Enter **Subnet name** (e.g., `SALES_SRV`)
4. Choose **Availability Zone**
5. Enter **Subnet CIDR** (must be a subset of VPC CIDR)

   * Example:

     * VPC: `10.10.0.0/16`
     * Subnet: `10.10.10.0/24`
6. Click **Create**

---

## Subnet Rules (Memorize)

* Subnet CIDR **must be within VPC CIDR**
* Subnets **cannot span Availability Zones**
* Smaller CIDR = fewer usable IPs
* `/24` ≈ 254 usable IPs

**Exam Trap:**
A subnet cannot exist in multiple AZs

---

## Current State After These Steps

* VPC created
* Subnet(s) created
* No EC2 instances yet
* No routing or internet access yet

> This is the **network foundation only**

---

## AWS Direct Connect (High-Level)

* Dedicated **private connection** from on-premises → AWS
* Connects via **Virtual Private Gateway (VPG)**
* Used for:

  * Low latency
  * High bandwidth
  * Enterprise-grade reliability

**Exam Tip:**
Direct Connect ≠ Internet-based VPN

---

## Lab Mental Model

```text
VPC = Network
Subnet = Network segment (per AZ)
Routing + Gateways = Traffic control
```

---

## Topics This Will Connect To

* Route Tables
* Internet Gateway vs NAT Gateway
* Public vs Private Subnets
* Security Groups vs NACLs

---
