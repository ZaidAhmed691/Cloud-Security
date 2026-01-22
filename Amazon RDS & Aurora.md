# 1. Amazon RDS

## What is RDS?
- **Amazon RDS (Relational Database Service)** is a **managed SQL database service**
- AWS handles infrastructure, patching, backups, and availability
- You manage the database **engine, schema, and data**

## Supported Database Engines
- PostgreSQL
- MySQL
- MariaDB
- Oracle
- Microsoft SQL Server
- IBM DB2
- Amazon Aurora (AWS proprietary)

## Why Use RDS Instead of EC2?
RDS removes operational overhead:

- Automated provisioning and OS patching
- Automated backups and **Point-in-Time Restore**
- Built-in monitoring and performance metrics
- **Read replicas** for read scaling
- **Multi-AZ** for high availability and disaster recovery
- Controlled maintenance windows
- Easy scaling (vertical + horizontal)
- Storage backed by EBS

**Limitation:**  
- No SSH access to the underlying instances (fully managed service)

## Scaling in RDS
- **Vertical scaling:** change instance size
- **Horizontal scaling:** add read replicas (read traffic only)

## High Availability & DR
- **Multi-AZ deployment** creates a standby replica in another AZ
- Automatic failover in case of primary failure

## Backups & Recovery
- Automated backups enabled by default
- Supports **Point-in-Time Restore (PITR)**

## RDS Storage Auto Scaling (Exam Topic)
- Automatically increases storage when running low
- Prevents manual resizing and downtime

**Triggers when:**
- Free storage < 10%
- Condition lasts > 5 minutes
- ≥ 6 hours since last scaling
- Max storage limit is defined

**Use case:** unpredictable workloads  
**Supported by:** all RDS engines

---

# 2. RDS Read Replicas vs Multi-AZ (Exam-Critical)

## RDS Read Replicas
**Purpose:** Scale **read performance**

- Used to offload **read-heavy workloads**
- Supports up to **15 read replicas**
- Can be:
  - Same AZ
  - Cross-AZ
  - Cross-region
- Replication type: **Asynchronous**
- Data consistency: **Eventually consistent**
- Read replicas are **read-only**
- Can be **promoted to standalone databases**

### Key Characteristics
- Applications must **explicitly connect** to read replicas
- Used for:
  - Analytics
  - Reporting
  - Read scaling
- No `INSERT`, `UPDATE`, or `DELETE` allowed
- Only `SELECT` queries

### Networking & Cost
- **Same-region (even cross-AZ):** no replication cost
- **Cross-region:** replication traffic is **charged**

## RDS Multi-AZ
**Purpose:** **High availability & disaster recovery**

- Creates a **standby database** in another AZ
- Replication type: **Synchronous**
- One **single DNS endpoint**
- Automatic **failover** if primary fails
- Standby **cannot be used** for reads or writes

### Key Characteristics
- Zero application changes required
- No read scaling
- Protects against:
  - AZ failure
  - Instance failure
  - Storage failure
  - Network failure

## Read Replicas vs Multi-AZ (Quick Comparison)

| Feature | Read Replica | Multi-AZ |
|------|------------|---------|
| Primary purpose | Read scaling | High availability |
| Replication | Asynchronous | Synchronous |
| Read access | Yes | No |
| Write access | No | No |
| Failover | Manual (promote) | Automatic |
| DNS endpoint | Multiple | Single |
| Used for DR | ❌ | ✅ |

## Combined Usage (Important Exam Case)
- **Read Replicas can themselves be Multi-AZ**
- Common for:
  - High availability **and**
  - Read scalability

## Single-AZ → Multi-AZ Conversion
- **Zero downtime operation**
- Enable via **Modify → Enable Multi-AZ**
- No database shutdown required

### What Happens Internally
- Snapshot of primary DB is taken
- Snapshot restored into standby AZ
- Synchronous replication begins
- Standby kept fully in sync

## Exam Signals to Watch For
- **“Improve read performance” → Read Replicas**
- **“High availability / failover / DR” → Multi-AZ**
- **“Cross-region replication” → Read Replicas**
- **“No app changes allowed” → Multi-AZ**
- **“Eventually consistent reads” → Read Replicas**
- **“Single endpoint” → Multi-AZ**

---

# 3. Creating an Amazon RDS Database (What You Actually Need to Know)

## Database Creation Basics
- RDS is created via **Standard Create** or **Easy Create**
- **Standard Create** exposes all options → preferred for exam understanding
- Supported engines:
  - MySQL
  - PostgreSQL
  - MariaDB
  - Oracle
  - SQL Server
  - Aurora (AWS proprietary, covered separately)

## Engine & Templates
- Engine choice impacts:
  - Licensing
  - Features (IAM auth, Multi-AZ behavior, replicas)
- Templates:
  - **Free Tier** → auto-restricts options
  - **Dev/Test**
  - **Production** → full control (used for learning)

Exam tip:
- Template choice **does not change capabilities**, only defaults

## Availability & Durability
- **Single-AZ**
  - Cheapest
  - No automatic failover
- **Multi-AZ**
  - Synchronous standby
  - Automatic failover
- **Multi-AZ Cluster**
  - Newer architecture (Aurora-like)

Exam signal:
- “High availability / failover” → **Multi-AZ**

## Instance Configuration
- Instance class defines **CPU & memory**
- Categories:
  - Burstable (t3 / t4g)
  - Standard
  - Memory optimized
- Free tier commonly uses:
  - `db.t3.micro`

Exam tip:
- Vertical scaling = change instance class

## Storage Configuration
- Storage types:
  - gp2 / gp3 → general purpose
  - io1 / io2 → high IOPS (production)
- Storage Auto Scaling:
  - Automatically increases storage
  - Triggered when:
    - <10% free space
    - Sustained for ~5 minutes
    - 6 hours since last resize
- Requires setting **maximum storage limit**

Exam signal:
- “Unpredictable storage growth” → **Enable storage autoscaling**

## Networking & Connectivity
- RDS runs **inside a VPC**
- Options:
  - Connect to EC2 automatically (AWS manages SG rules)
  - Manual VPC + subnet group + security groups
- Public access:
  - **Yes** → accessible from internet (labs/testing)
  - **No** → private access only (production)

Exam tip:
- Production databases are usually **NOT publicly accessible**

## Security Groups
- Database port must be allowed:
  - MySQL → 3306
  - PostgreSQL → 5432
- Best practice:
  - Allow inbound **only from app security groups**
- For labs:
  - Allow from your IP for testing

Exam signal:
- “Cannot connect to RDS” → check **security group & public access**

## Authentication Methods
- Username & password (default)
- IAM Database Authentication
- Kerberos (enterprise use)

Exam tip:
- IAM auth removes need for DB passwords in apps

## Backups & Maintenance
- Automated backups:
  - Retention: 1–35 days
  - Enables **Point-in-Time Restore**
- Backup window & maintenance window:
  - Can be defined or left to AWS
- Deletion protection:
  - Prevents accidental deletion

Exam signal:
- “Restore to a specific time” → **Automated backups enabled**

## Monitoring
- Built-in CloudWatch metrics:
  - CPU utilization
  - Database connections
  - Read/write latency
- Enhanced monitoring:
  - OS-level metrics
  - Higher granularity

Exam tip:
- Monitoring does **not** require installing agents

## Read Replicas & Snapshots
- Read replicas:
  - Created from existing DB
  - Used for read scaling
- Snapshots:
  - Manual or automated
  - Can be copied cross-region
  - Used for backup & migration

Exam signal:
- “Analytics workload impacting production DB” → **Read Replica**

## Deletion Workflow (Common Lab Gotcha)
- Must disable **Deletion Protection**
- Optional final snapshot
- Deletion is irreversible

## What the Exam Cares About (Summary)
- Single-AZ vs Multi-AZ
- Read Replicas vs Multi-AZ
- Storage autoscaling behavior
- Public vs private access
- Backup retention & PITR
- Security group access patterns

---

# 4. Amazon RDS Custom (Exam Summary)

## What RDS Custom Is
- **RDS Custom = managed database + OS access**
- AWS still manages:
  - Provisioning
  - Scaling
  - Backups
- **You get access to the underlying EC2 instance and OS**

## Supported Engines (Very Important)
RDS Custom is available **ONLY** for:
- **Oracle**
- **Microsoft SQL Server**

Exam signal:
- “Need OS access on RDS” → **RDS Custom (Oracle / SQL Server)**

## Why RDS Custom Exists
Standard RDS:
- No SSH
- No OS-level access
- No deep DB customization

RDS Custom:
- Full admin access to:
  - Operating system
  - Database engine
- Allows:
  - Custom patches
  - Native DB features
  - Internal configuration changes

## Access & Customization
With RDS Custom you can:
- SSH into the underlying EC2 instance
- Use **SSM Session Manager**
- Modify OS and database settings directly

This breaks the “black box” limitation of normal RDS.

## Automation Mode (Critical Concept)
- RDS Custom normally runs with **automation enabled**
- Before making changes:
  - **Disable automation mode**
  - Prevents AWS from:
    - Applying patches
    - Restarting services
    - Overwriting your changes

Exam signal:
- “Prevent AWS from interfering with custom DB changes” → **Disable automation mode**

## Risk & Best Practice
- Because you can modify the OS:
  - You can **break the database**
- **Strongly recommended**:
  - Take a **manual snapshot** before customization
- If broken:
  - Restore from snapshot

Exam signal:
- “Recover from bad OS changes” → **Restore snapshot**

## RDS vs RDS Custom (One-Line Comparison)

| Feature | RDS | RDS Custom |
|------|----|-----------|
| OS access | ❌ No | ✅ Yes |
| SSH / SSM | ❌ No | ✅ Yes |
| AWS-managed automation | ✅ Full | ⚠️ Can be disabled |
| Supported engines | Many | Oracle, SQL Server |
| Custom patches | ❌ No | ✅ Yes |

## When to Use RDS Custom
Use **RDS Custom** when:
- Vendor requires OS-level changes
- You need native Oracle / SQL Server features
- Compliance requires custom configuration

Do **NOT** use RDS Custom:
- For MySQL / PostgreSQL
- If you want zero operational responsibility

## Exam Keywords to Watch For
- “OS-level access”
- “SSH into RDS”
- “Custom database patches”
- “Oracle / SQL Server customization”
- “Disable automation mode”

All → **RDS Custom**
