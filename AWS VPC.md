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

Below is **GitHub-friendly, exam-style Markdown**, numbered exactly as requested and easy to re-paste into your notes repo.

---

# 2. AWS VPC DHCP Option Sets

> **Goal:** Control DHCP configuration parameters for instances inside a VPC.

---

## How DHCP Works in AWS VPC

* AWS **automatically handles IP address assignment**
* IP ranges are defined by **subnet CIDR**
* You **do NOT define DHCP IP pools manually** like on-prem

👉 In AWS, you only configure **DHCP options**, not address ranges

---

## What Is a DHCP Option?

* A **DHCP option** is a configuration parameter delivered via DHCP
* Devices receive:

  * IP address (automatic)
  * Subnet mask
  * Default gateway
  * **Additional options** (DNS, domain name, NTP, etc.)

---

## DHCP Option Sets

* A **DHCP Option Set** is a collection of DHCP configuration parameters
* Created separately, then **associated with a VPC**
* A VPC can have **only one DHCP option set at a time**

---

## DHCP Options You Can Configure

1. **Domain Name**

   * Example: `sales.mycompany.local`
2. **Domain Name Servers (DNS)**

   * Example: `8.8.8.8` (Google DNS)
3. **NTP Servers**

   * Example: `10.10.10.37`
4. **NetBIOS Name Servers** (legacy)
5. **NetBIOS Node Type** (legacy)

📌 You do **not** need to configure all options

---

## Steps to Create a DHCP Option Set

1. Go to **VPC Console**
2. Select **DHCP Option Sets**
3. Click **Create DHCP option set**
4. Enter **Name tag** (e.g., `SALES`)
5. Configure required options:

   * Domain name
   * DNS servers
   * NTP servers (optional)
6. Click **Create**

---

## After Creation

* DHCP option set exists independently
* Must be **associated with a VPC** to take effect
* Applies to **all instances launched in that VPC**
* Existing instances may require **renew/restart** to pick up changes

---

## Key Rules (Exam-Focused)

* Subnet CIDR defines IP range, **not DHCP**
* DHCP Option Sets define **configuration only**
* One DHCP Option Set per VPC
* Multiple option sets may exist, but **only one active per VPC**

---

## Common Exam Traps

* ❌ “Define DHCP IP pool in AWS” → **Wrong**
* ❌ “DHCP configured per subnet” → **Wrong**
* ✅ “DHCP options configured at VPC level” → **Correct**

---

## Lab Mental Model

```text
Subnet = IP range
DHCP Option Set = configuration parameters
VPC = attachment point
```

---

## When You’d Use Multiple DHCP Option Sets

* Different environments (dev / prod)
* Different DNS or NTP requirements
* Multiple VPCs with different policies

---

## What This Connects To Next

* VPC associations
* EC2 instance networking
* Route tables & gateways
* Hybrid networking (VPN / Direct Connect)

---

Below is **GitHub-friendly, exam-style Markdown**, numbered exactly as requested and ready to paste into your notes repo.

---

# 3. AWS Elastic IPs (EIP)

> **Goal:** Provide a stable public IPv4 address for internet-facing resources in a VPC.

---

## What Is an Elastic IP (EIP)?

* An **Elastic IP** is a **public IPv4 address**
* Allocated from the **AWS region** where your VPC exists
* Used to make instances **reachable from the internet**

---

## Why “Elastic”?

* Can be **re-associated** with another instance
* Useful for:

  * Instance replacement
  * Failover
  * Maintenance without DNS changes

📌 Elastic ≠ temporary

---

## Key Characteristics (High Yield)

* **Public IP** (not private ranges like `10.x`, `172.16–31.x`, `192.168.x`)
* **Allocated to your AWS account**
* **Remains allocated until released**
* **Charged while allocated**, even if unused

---

## Cost Rule (Very Important)

* You are **charged for an EIP when it is not associated**
* Unused EIPs = **wasted money**

**Exam Tip:**
AWS charges to discourage public IPv4 hoarding

---

## How EIPs Are Used

* EIPs are **associated with an Elastic Network Interface (ENI)**
* ENI attaches to:

  * EC2 instance
  * Other supported services

```text
EIP → ENI → EC2 Instance
```

---

## Region Limitation

* EIPs are **region-specific**
* Can be moved:

  * Between instances
  * **Only within the same region**

❌ Cannot move an EIP across regions

---

## When You Need an EIP

* Internet-facing servers
* Static public IP requirement
* Legacy apps needing fixed IPs
* External systems whitelisting IPs

---

## Steps to Allocate an Elastic IP

1. Go to **VPC Console**
2. Select **Elastic IPs**
3. Click **Allocate Elastic IP address**
4. Choose:

   * **Amazon pool** (most common)
   * Or **Customer-owned IP**
5. Click **Allocate**

---

## After Allocation

* EIP is:

  * Assigned to your account
  * **Not attached to anything yet**
* Starts **incurring cost if unused**

---

## Associate an Elastic IP

* Associate EIP with:

  * An **ENI**
  * Or directly to an **EC2 instance**
* This enables **public internet access**

---

## How to Release an Elastic IP

1. Ensure it is **not associated** with any instance
2. Select the EIP
3. **Actions → Release Elastic IP addresses**
4. Confirm release

📌 Always verify before releasing

---

## Common Exam Traps

* ❌ “EIPs are free” → **Wrong**
* ❌ “EIPs are private IPs” → **Wrong**
* ❌ “EIPs can move across regions” → **Wrong**
* ✅ “Charged when allocated and unused” → **Correct**

---

## Lab Mental Model

```text
Private IP → internal communication
Public IP / EIP → internet access
EIP = stable public identity
```

---

## What This Connects To Next

* Internet Gateways
* Route Tables
* Public vs Private Subnets
* NAT Gateway vs EIP usage

---
