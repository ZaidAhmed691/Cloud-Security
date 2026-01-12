# 1. EC2 Instance Types

## Overview
- EC2 instances come in **different types** optimized for **different workloads**
- Categories are based on **compute, memory, storage, and networking**
- The AWS EC2 Instance Types page is the **source of truth** for specs and pricing
- Instance families evolve over time

## EC2 Naming Convention
Example: `m5.2xlarge`

- `m` → Instance class (workload optimization)
- `5` → Instance generation (newer hardware)
- `2xlarge` → Instance size  
  → Larger size = more vCPU, more RAM, higher performance

## General Purpose Instances
- Balanced **CPU, memory, and networking**
- Suitable for **diverse workloads**
- Common use cases:
  - Web servers
  - Application servers
  - Code repositories
- Example:
  - `t2.micro` (Free Tier, used in labs)

## Compute Optimized Instances
- Designed for **CPU-intensive workloads**
- Prioritize **high compute performance**
- Common use cases:
  - Batch processing
  - Media transcoding
  - High-performance web servers
  - High Performance Computing (HPC)
  - Machine learning
  - Gaming servers
- Instance family:
  - `C` series (e.g., `c5`, `c6`)

## Memory Optimized Instances
- Designed for **large datasets in RAM**
- Emphasis on **fast memory access**
- Common use cases:
  - In-memory databases
  - Relational and NoSQL databases
  - Distributed caching (e.g., ElastiCache)
  - Business Intelligence (BI)
  - Real-time big data processing
- Instance families:
  - `R` series
  - `X1`, High Memory, `Z1`

## Storage Optimized Instances
- Designed for **high disk I/O and local storage**
- Best for **data-intensive workloads**
- Common use cases:
  - OLTP systems
  - Relational and NoSQL databases
  - In-memory database cache (e.g., Redis)
  - Data warehousing
  - Distributed file systems
- Instance families:
  - `I`, `D`, `H1`

## Conceptual Instance Comparison
- `t2.micro`
  - 1 vCPU
  - 1 GB RAM

- `r5.16xlarge`
  - Memory-heavy
  - Very large RAM footprint

- `c5.d.4xlarge`
  - High CPU
  - Lower memory relative to RAM-optimized instances

→ Instance families prioritize **different resource dimensions**

## Useful Tools
- **ec2instances.info**
  - Compare all EC2 instances
  - View vCPU, memory, pricing, and bandwidth
  - Filter and sort instances easily

## Exam Takeaways
- Focus on **workload → instance type mapping**
- Exact specs do **not** need memorization
- Key associations:
  - General Purpose → balanced workloads
  - Compute Optimized → CPU-heavy workloads
  - Memory Optimized → RAM-heavy workloads
  - Storage Optimized → high I/O workloads
- EC2 naming conventions are **conceptually tested**

---

