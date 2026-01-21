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
