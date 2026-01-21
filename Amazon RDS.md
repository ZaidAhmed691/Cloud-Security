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

