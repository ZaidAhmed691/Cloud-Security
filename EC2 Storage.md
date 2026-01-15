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

---

# 3. EBS Snapshots

## Overview
- **EBS Snapshots** are **point-in-time backups** of EBS volumes
- Used for:
  - Backup
  - Disaster recovery
  - Moving data across AZs or Regions
- Snapshots are stored in **Amazon S3** (managed by AWS)

## Creating Snapshots
- Snapshot can be taken:
  - While the volume is attached and in-use
  - Detaching the volume is **recommended** for consistency
- Snapshot captures the state of the volume at that moment

## Moving Data Across AZs and Regions
- EBS volumes are **AZ-bound**
- Snapshots are **not AZ-bound**
- Workflow:
  - Take snapshot of volume in AZ-A
  - Restore snapshot in AZ-B or another Region
- This is the **only way** to move EBS data across AZs

## Snapshot Archive Tier
- Feature: **EBS Snapshot Archive**
- Moves snapshot to a **low-cost archive tier**
- Cost savings:
  - Up to **75% cheaper**
- Trade-off:
  - Restore time: **24–72 hours**
- Best for:
  - Long-term backups
  - Rarely accessed snapshots

## Recycle Bin for EBS Snapshots
- Protects against **accidental deletion**
- Deleted snapshots go to **Recycle Bin** instead of permanent deletion
- Retention period:
  - Configurable from **1 day to 1 year**
- Allows snapshot recovery during retention window

## Fast Snapshot Restore (FSR)
- Ensures **fully initialized snapshots**
- Eliminates:
  - Latency on first read
- Useful for:
  - Large snapshots
  - Time-sensitive restores
- Downside:
  - **Expensive**
- Use only when fast restore is critical

## Key Takeaways
- Snapshots are **point-in-time backups**
- Used to move volumes across AZs / Regions
- Archive tier = cheaper, slower restore
- Recycle Bin = safety net for deletion
- Fast Snapshot Restore = performance boost at higher cost
- Exam focus is on **use cases**, not pricing details

---

# 4. EBS Snapshots – Hands-On & Advanced Features

## Creating an EBS Snapshot
- From EC2 → **Volumes**
- Select an EBS volume (e.g., 2 GB `gp2`)
- Actions → **Create snapshot**
- Add a description (e.g., `DemoSnapshots`)
- Snapshot creation is asynchronous
- Once completed:
  - Status = **Completed**
  - Snapshot is fully available

## Viewing Snapshots
- EC2 → **Snapshots**
- Displays:
  - Snapshot ID
  - Status
  - Source volume
  - Size
  - Region

## Copying Snapshots Across Regions
- Snapshots can be **copied to another Region**
- Use case:
  - Disaster Recovery (DR)
  - Cross-region backups
- Workflow:
  - Snapshot → Actions → **Copy snapshot**
  - Choose destination Region
- Enables regional data protection

## Restoring a Volume from a Snapshot
- Snapshot → Actions → **Create volume from snapshot**
- Choose:
  - Volume type (e.g., `gp2`)
  - Size
  - **Target Availability Zone** (can differ from original)
  - Optional encryption and tags
- Result:
  - New EBS volume created from snapshot
  - Confirms snapshots allow **cross-AZ volume recovery**

## Snapshot Storage Tiers
- Default tier:
  - **Standard**
- Optional:
  - **Archive tier**
    - Lower cost
    - Restore time: **24–72 hours**
- Suitable for long-term backups

## Recycle Bin for EBS Snapshots
- Protects against **accidental deletion**
- Requires a **Retention Rule**
- Steps:
  - EC2 → Recycle Bin → Create retention rule
  - Apply to:
    - EBS Snapshots
  - Retention:
    - 1 day to 1 year
  - Rule lock:
    - Optional (can prevent deletion of the rule)

## Deleting and Recovering Snapshots
- Delete a snapshot:
  - Snapshot is removed from Snapshots list
- With Recycle Bin enabled:
  - Snapshot appears in **Recycle Bin → Resources**
- Recovery:
  - Select snapshot → **Recover**
  - Snapshot is restored to EC2 Snapshots

## Key Takeaways
- Snapshots are **point-in-time backups**
- Snapshots are **Region-level**, not AZ-bound
- Can:
  - Copy snapshots across Regions
  - Restore volumes into different AZs
- Archive tier reduces cost with slower restore
- Recycle Bin provides snapshot recovery
- Common exam focus:
  - DR strategy
  - Cross-AZ migration
  - Accidental deletion protection

---

# 5. Amazon Machine Images (AMI)

## Overview
- **AMI (Amazon Machine Image)** defines how an EC2 instance is launched
- It is a **template** containing:
  - Operating system
  - Software configuration
  - Pre-installed packages and tools
- AMIs enable **faster boot and configuration** since software is prepackaged

## What an AMI Contains
- Operating system (e.g., Linux, Windows)
- Installed applications and dependencies
- Configuration settings
- Monitoring and agents (if added)
- Backed by **EBS snapshots** (created automatically)

## Types of AMIs
- **AWS-provided AMIs**
  - Public AMIs maintained by AWS
  - Example: Amazon Linux 2
- **Custom AMIs**
  - Built and maintained by you
  - Optimized for your workloads
  - Faster and more consistent launches
- **AWS Marketplace AMIs**
  - Built and sold by third-party vendors
  - Often include licensed software
  - Can save setup and configuration time

## Regional Scope
- AMIs are **region-specific**
- You can:
  - Create an AMI in one region
  - **Copy the AMI to other regions**
- Enables global deployment using the same image

## AMI Creation Workflow
- Launch an EC2 instance
- Customize it (install software, configure OS)
- **Stop the instance** (recommended for data integrity)
- Create an AMI from the instance
  - AWS creates EBS snapshots automatically
- Launch new EC2 instances from the AMI

## AMIs and Availability Zones
- AMIs can be used to launch instances in **any AZ within the same region**
- Example workflow:
  - Create instance in `us-east-1a`
  - Create AMI
  - Launch identical instance in `us-east-1b`

## Use Cases
- Faster instance provisioning
- Consistent environments (dev / test / prod)
- Disaster recovery and scaling
- Sharing or selling software via AWS Marketplace

## Key Takeaways
- AMI = blueprint for EC2 instances
- Prepackaged software = faster launches
- AMIs are region-bound but copyable
- Custom AMIs improve consistency and speed
- Marketplace AMIs provide ready-made solutions

---

# 6. Amazon Machine Images (AMI) – Hands-On Practice

## Launching an Instance to Create an AMI
- Launch a new EC2 instance:
  - AMI: Amazon Linux 2
  - Instance type: `t2.micro`
  - Key pair: any
  - Security group: existing (e.g., `launch-wizard-1`)
- Configure **User Data**:
  - Install Apache (`httpd`)
  - Start the service
  - **Do NOT create the index file yet**
- Purpose:
  - Prepare a base instance with software preinstalled

## Waiting for User Data Execution
- Instance may show **Running** before setup is complete
- User Data runs **only on first boot**
- Apache setup can take 1–2 minutes
- Once complete:
  - Access the public IPv4 via HTTP
  - Default Apache test page confirms installation

## Creating an AMI from the Instance
- EC2 → Instance → Right-click
- Image and templates → **Create image**
- Provide:
  - Image name (e.g., `demo-image`)
- Leave default settings
- Create image:
  - Instance is snapshotted
  - EBS snapshots are created automatically
- AMI status:
  - Initially **Pending**
  - Then **Available**

## Launching an Instance from the Custom AMI
- EC2 → Launch instance
- Choose **My AMIs**
- Select the custom AMI (`demo-image`)
- Configure:
  - Key pair (optional)
  - Security group (existing)
- Add **User Data**:
  - Only create/update the index file
  - Do NOT reinstall Apache (already in the AMI)

## Verifying the AMI Benefits
- Launch instance from AMI
- Access public IPv4 via browser
- Result:
  - Custom webpage loads quickly
  - No delay for Apache installation
- Demonstrates:
  - Faster boot time
  - Preinstalled software reuse

## Why AMIs Are Powerful
- Avoid repeated software installation
- Faster scaling and recovery
- Consistent environments
- Ideal for:
  - Web servers
  - Security agents
  - Preconfigured app stacks

## Cleanup
- Terminate:
  - Original EC2 instance
  - Instance launched from AMI
- AMI remains available for reuse

## Key Takeaways
- AMIs capture the full instance state
- User Data runs once, on first boot
- Custom AMIs significantly reduce startup time
- AMIs enable repeatable, consistent EC2 deployments
- Common exam focus:
  - Faster boot with preinstalled software
  - Reusability across AZs within a region

---

# 7. EC2 Instance Store (Local Storage)

## Overview
- **EC2 Instance Store** is **local storage physically attached** to the EC2 host machine
- Provides **very high I/O performance**
- Storage is **ephemeral (temporary)**

## What is EC2 Instance Store?
- Hardware disk attached directly to the physical server
- Not network-based (unlike EBS)
- Available only on **specific EC2 instance types**
- Used when **extreme disk performance** is required

## Key Characteristics
- **Very high IOPS & throughput**
  - Millions of IOPS possible
  - Much faster than EBS
- **Low latency**
  - Direct physical connection
- **Ephemeral storage**
  - Data is lost if:
    - Instance is stopped
    - Instance is terminated
    - Underlying hardware fails

## Durability & Risk
- Data is **NOT durable**
- If the host machine fails:
  - Data is permanently lost
- AWS does **not** replicate instance store data
- You are responsible for:
  - Backups
  - Replication
  - Data recovery strategy

## Comparison with EBS
- **Instance Store**
  - Local, physical disk
  - Extremely high performance
  - Temporary
- **EBS**
  - Network-attached
  - Durable
  - Lower performance than instance store
  - Suitable for long-term storage

## Common Use Cases
- Cache
- Buffer storage
- Temporary files
- Scratch data
- High-performance workloads that can tolerate data loss

## What NOT to Use It For
- Databases requiring durability
- Long-term data storage
- Critical application data

## Exam Tip
- If you see:
  - **Very high disk performance**
  - **Local hardware storage**
  - **Temporary / ephemeral data**
- Think: **EC2 Instance Store**

## Key Takeaways
- Instance Store = fastest EC2 storage
- Data loss on stop/terminate
- No durability guarantees
- Best for temporary, high-speed workloads
- EBS is preferred for persistent storage
