# 1. AWS RDS Overview

So let's get started with an overview of AWS RDS.

RDS stands for **Relational Database Service**, and what it means is that it's a **managed database service** for databases that use **SQL** as a query language.

SQL is a structured language used to query databases. It's very well adapted and runs on many database engines.

With RDS, you can create databases in the cloud that are **fully managed by AWS**, and you get many operational benefits.

## Supported Database Engines

Amazon RDS supports the following database engines:

- PostgreSQL  
- MySQL  
- MariaDB  
- Oracle  
- Microsoft SQL Server  
- IBM DB2  
- Amazon Aurora (AWS proprietary database engine)

These are important to remember for the exam.

## Why Use RDS Instead of Running a Database on EC2?

You *can* run your own database engine on an EC2 instance, but RDS provides major advantages because it is a **managed service**.

With RDS, AWS handles:

- Automated database provisioning  
- Operating system patching  
- Continuous automated backups  
- **Point-in-Time Restore** (restore to a specific timestamp)  
- Monitoring dashboards and performance metrics  
- Read replicas to improve read performance  
- Multi-AZ deployments for high availability and disaster recovery  
- Maintenance windows for upgrades  
- Vertical scaling (changing instance size)  
- Horizontal scaling (adding read replicas)  

RDS storage is backed by **EBS**, which you already know.

One important limitation:
- You **cannot SSH** into RDS instances because AWS manages the underlying infrastructure.

This tradeoff is usually worth it, since AWS handles many complex operational tasks for you.

## RDS Storage Auto Scaling

A feature that can appear in the exam is **RDS Storage Auto Scaling**.

When creating an RDS database, you must define an initial storage size (for example, 20 GB). If your database workload grows and you start running out of free space, RDS can **automatically increase storage** for you if this feature is enabled.

This avoids downtime or manual operations to resize storage.

### How Storage Auto Scaling Works

Storage auto scaling will trigger when:

- Free storage drops below **10%**
- Low storage condition lasts for **more than 5 minutes**
- At least **6 hours** have passed since the last storage modification
- A **maximum storage threshold** has been defined

When these conditions are met, RDS automatically increases storage up to the maximum limit you configured.

### Why This Is Useful

- Ideal for **unpredictable workloads**
- Prevents database outages due to running out of storage
- Supported by **all RDS database engines**

This makes RDS much easier to operate at scale.

---

