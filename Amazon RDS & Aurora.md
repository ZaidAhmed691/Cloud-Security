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

---

# 5. Amazon Aurora (Exam Summary)

## What Aurora Is
- **AWS proprietary relational database**
- **MySQL- and PostgreSQL-compatible**
  - Same drivers
  - Minimal application changes
- **Cloud-optimized**, not open source

Exam signal:
- “AWS proprietary MySQL/Postgres-compatible DB” → **Aurora**

## Performance & Cost
- ~**5× faster than MySQL on RDS**
- ~**3× faster than PostgreSQL on RDS**
- ~**20% more expensive than RDS**
- More cost-effective **at scale** due to efficiency

## Storage (Very Important)
- **Starts at 10 GB**
- **Automatically scales up to 256 TB**
- No manual disk management
- No downtime for storage growth

Exam signal:
- “Automatic storage scaling without downtime” → **Aurora**

## High Availability Architecture
- **6 copies of data across 3 AZs**
- Write requires **4/6 copies**
- Read requires **3/6 copies**
- Survives:
  - AZ failure
  - Storage failure
- **Self-healing storage**
- Data striped across **hundreds of volumes**

Key takeaway:
- Aurora storage is **distributed, replicated, and self-healing by default**

## Writer & Read Replicas
- **1 writer (master)**
- **Up to 15 read replicas**
- Replication is **very fast** (sub-10 ms lag)
- Any replica can be **promoted to writer**

Failover:
- **< 30 seconds** (faster than RDS Multi-AZ)

Exam signal:
- “Fast failover + read scaling” → **Aurora**

## Endpoints (Extremely Important for Exam)

### Writer Endpoint
- DNS name
- Always points to **current writer**
- Handles failover transparently
- Applications **never connect directly to instances**

### Reader Endpoint
- DNS name
- Load balances **connections** across read replicas
- Supports **auto-scaling replicas**

Important:
- Load balancing is **connection-level**, not query-level

Exam signal:
- “Reader endpoint” or “Writer endpoint” → **Aurora**

## Read Scaling
- Up to **15 read replicas**
- Supports **cross-region replication**
- **Auto-scaling** of read replicas supported

Use cases:
- Heavy read workloads
- Global read access

## Backup & Recovery
- Continuous backups
- Point-in-time restore
- **Backtrack feature**
  - Restore to a specific second in time
  - Does **not rely on snapshots**
  - Fast rewind/forward in time

Exam signal:
- “Restore without snapshot” → **Aurora Backtrack**

## Operations & Management
Fully managed by AWS:
- Automated patching (zero downtime)
- Monitoring & metrics
- Routine maintenance
- Security & compliance
- No OS access required

## Aurora vs RDS (Quick Comparison)

| Feature | RDS | Aurora |
|------|----|--------|
| Storage scaling | Manual / Auto (limited) | Fully automatic |
| Replication | Slower | Sub-10 ms |
| Failover | Minutes | < 30 sec |
| Read replicas | Yes | Yes (faster, more scalable) |
| Architecture | Instance-based | Distributed storage |
| Cost | Lower | ~20% higher |

## When to Choose Aurora
Choose **Aurora** when:
- High availability is critical
- Fast failover is required
- Large read scaling is needed
- You want minimal operational effort
- Storage growth must be automatic

Do NOT choose Aurora when:
- You want lowest cost at small scale
- You need OS-level access (use RDS Custom)

## Exam Keywords to Watch For
- “Writer endpoint”
- “Reader endpoint”
- “6 copies across 3 AZs”
- “Automatic storage scaling”
- “Fast failover”
- “Aurora MySQL / PostgreSQL”

All → **Amazon Aurora**

---

# 6. Amazon Aurora – Hands-on Creation, Configuration, and Cleanup

## Overview
- Amazon Aurora is a fully managed, high-performance relational database compatible with MySQL and PostgreSQL.
- This hands-on exercise incurs cost and is not part of the Free Tier.
- Best practice: understand the configuration options without creating the database if cost is a concern.

## Database Creation
- Creation method: Standard Create
- Engine options:
  - Aurora MySQL-compatible
  - Aurora PostgreSQL-compatible
- Selected engine: Aurora MySQL
- Engine version:
  - Default version suggested by the AWS Console (e.g., 3.04.1)
  - Filters available:
    - Global Database support
    - Parallel Query support
    - Serverless v2 support

## Template and Credentials
- Template: Production
  - Enables full configuration control
- DB Cluster Identifier: `database-two`
- Master username: `admin`
- Password: User-defined

## Storage Configuration
- Aurora Standard
  - Cost-effective for moderate workloads
- Aurora IO-Optimized
  - Optimized for high read/write IOPS workloads

## Instance Configuration
- Instance class options:
  - Burstable
  - Memory-optimized
  - Previous generation instances (optional)
- Selected instance: `db.t3.medium`
- Aurora Serverless v2 (optional):
  - Uses Aurora Capacity Units (ACU)
  - Configure minimum and maximum ACUs
  - Automatic compute scaling

## Availability and Durability
- Aurora Replica enabled
  - Creates a reader instance in a different Availability Zone
  - Benefits:
    - High availability
    - Faster failovers
    - Improved read performance
  - Increases cost due to multi-AZ deployment

## Networking and Connectivity
- Network type: IPv4
  - Dual-stack (IPv4 + IPv6) supported if VPC allows
- VPC: Default VPC
- Public access: Enabled (demo use case)
- Security Group:
  - New SG created: `demo-database-aurora`

## Additional Configuration
- Database port: `3306` (MySQL)
- Local Write Forwarding
  - Writes sent to reader are forwarded to the writer automatically
- Authentication options:
  - Password-based
  - IAM Database Authentication
  - Kerberos Authentication
- Enhanced Monitoring: Disabled
- Initial database name: `mydb`
- Backup retention: 1 day
- Encryption: Supported
- Deletion protection: Optional

## Cost Awareness
- AWS Console displays estimated monthly cost before creation
- Critical to review costs prior to provisioning

## Aurora Architecture
- Regional cluster consists of:
  - One Writer instance
  - One Reader instance
  - Instances distributed across different Availability Zones
- Endpoints:
  - Writer Endpoint: Always routes to the active writer
  - Reader Endpoint: Load-balances read traffic across readers
  - Each instance also has its own dedicated endpoint
- Best practice: Applications should use cluster endpoints instead of instance endpoints

## Scaling Features
- Manual Read Replica Scaling
  - Add additional readers to increase read throughput
- Read Replica Auto Scaling
  - Policy-based scaling using metrics such as:
    - Average CPU utilization
    - Number of connections
  - Example configuration:
    - Target utilization: 60%
    - Minimum replicas: 1
    - Maximum replicas: 15
- Cross-region read replicas
  - Used for global read access and disaster recovery

## Global Database
- Available only if:
  - Aurora version supports Global Database
  - Instance size is compatible (e.g., Large or higher)
- Enables:
  - Cross-region replication
  - Low-latency global reads

## Database Deletion and Cleanup
- Aurora clusters cannot be deleted directly
- Required steps:
  1. Delete the Reader instance (confirm with `delete me`)
  2. Delete the Writer instance (confirm with `delete me`)
  3. Delete the Aurora cluster
- Cluster deletion is possible only after all instances are removed

## Key Takeaways
- Aurora provides:
  - High performance
  - Multi-AZ availability
  - Reader/writer separation
  - Automatic and manual scaling
  - Serverless capabilities
  - Global database support
- Powerful production-grade database, but cost-sensitive
- Best suited for scalable, high-availability workloads

---

# 7. Amazon Aurora – Advanced Concepts for the Exam

## Replica Auto Scaling
- Used to automatically scale **read replicas** based on load
- Typical setup:
  - 1 Writer instance (writer endpoint)
  - Multiple Reader instances (reader endpoint)
- Scenario:
  - Increased read traffic causes high CPU usage on reader instances
  - Replica Auto Scaling adds new Aurora read replicas automatically
  - Reader endpoint automatically includes new replicas
- Benefits:
  - Distributes read traffic more evenly
  - Reduces CPU utilization
  - Improves read performance without manual intervention

## Custom Endpoints
- Used when read replicas have **different instance sizes**
  - Example:
    - Smaller instances: `db.r3.large`
    - Larger instances: `db.r5.2xlarge`
- Purpose:
  - Create a **custom endpoint** that targets a specific subset of replicas
- Common use case:
  - Route analytical or heavy queries to larger instances
  - Keep lightweight reads on smaller replicas
- Key behavior:
  - Reader endpoint still exists but is typically not used
  - Applications explicitly connect to custom endpoints
- Best practice:
  - Create multiple custom endpoints for different workloads

## Aurora Serverless
- Fully managed, on-demand Aurora capacity
- Key characteristics:
  - Automatic database instantiation
  - Automatic scaling based on actual usage
  - No capacity planning required
- Ideal workloads:
  - Infrequent
  - Intermittent
  - Unpredictable traffic patterns
- Cost model:
  - Pay per second for Aurora capacity used
  - Can be significantly more cost-effective
- Architecture:
  - Client connects to an Aurora-managed proxy fleet
  - Backend Aurora instances scale up or down automatically
  - No fixed instances provisioned in advance

## Aurora Global Database
- Designed for:
  - Low-latency global reads
  - Disaster recovery
- Two approaches:
  - Cross-Region Read Replica (simpler)
  - Aurora Global Database (recommended)
- Global Database characteristics:
  - One **primary region** (reads and writes)
  - Up to **10 secondary read-only regions**
  - Up to **16 read replicas per secondary region**
  - Cross-region replication lag: **< 1 second**
- Disaster Recovery:
  - Secondary region can be promoted to read-write
  - Recovery Time Objective (RTO): **< 1 minute**
- Exam keyword:
  - “Replicates data across regions in less than one second” → Use **Aurora Global Database**
- Example:
  - Primary: `us-east-1`
  - Secondary: `eu-west-1`
  - Applications read locally, fail over if primary fails

## Aurora Machine Learning
- Enables ML-based predictions directly via **SQL queries**
- Seamless integration between Aurora and AWS ML services
- Supported services:
  - Amazon SageMaker
    - Any custom ML model
  - Amazon Comprehend
    - Sentiment analysis
- Key points:
  - No ML expertise required
  - Predictions accessed through SQL
- Common use cases:
  - Fraud detection
  - Product recommendations
  - Ads targeting
  - Sentiment analysis
- Architecture flow:
  - Application runs SQL query
  - Aurora sends data to ML service
  - ML service returns prediction
  - Aurora returns result as SQL query output

## Babelfish for Aurora PostgreSQL
- Enables Aurora PostgreSQL to understand **T-SQL**
- Purpose:
  - Migrate from Microsoft SQL Server to Aurora PostgreSQL
  - Avoid rewriting application code
- How it works:
  - Applications continue using:
    - SQL Server client driver
    - T-SQL language
  - Babelfish translates T-SQL into PostgreSQL-compatible queries
- Benefits:
  - Minimal to no application code changes
  - Simplifies database migrations
- Migration tools:
  - AWS Schema Conversion Tool (SCT)
  - AWS Database Migration Service (DMS)
- Key takeaway:
  - Babelfish handles SQL translation
  - SCT and DMS handle schema and data migration

---

# 8. RDS and Aurora Backup, Restore, and Cloning

## RDS Automated Backups
- Managed automatically by the RDS service
- Performs:
  - Daily full backup during the backup window
  - Transaction log backups every 5 minutes
- Enables Point-in-Time Recovery (PITR):
  - Restore to any point up to **5 minutes before the current time**
- Retention period:
  - Configurable from **1 to 35 days**
- Disabling automated backups:
  - Set retention period to **0**
  - This fully disables automated backups

## RDS Manual DB Snapshots
- Manually triggered by the user
- Retention:
  - Kept **as long as you want**
- Key difference from automated backups:
  - Automated backups expire
  - Manual snapshots do not expire unless deleted

## Cost-Saving Backup Strategy (Exam Tip)
- Use case:
  - Database is used infrequently (e.g., a few hours per month)
- Recommended approach:
  1. Use the database when needed
  2. Take a manual snapshot
  3. Delete the RDS database
  4. Retain the snapshot (cheaper storage)
  5. Restore the snapshot when needed again
- Advantage:
  - Snapshot storage costs less than running an idle RDS instance

## Aurora Backups
- Similar to RDS but with key differences
- Automated backups:
  - Enabled by default
  - Retention: **1 to 35 days**
  - **Cannot be disabled**
- Supports Point-in-Time Recovery within the retention window
- Manual DB snapshots:
  - User-triggered
  - Retained indefinitely
- Key takeaway:
  - Aurora and RDS backups behave similarly
  - Aurora always has automated backups enabled

## Restore Options Overview
- Restoring from:
  - Automated backups
  - Manual snapshots
- Behavior:
  - Restore always creates a **new database**
  - Original database remains unchanged

## Restoring RDS MySQL from Amazon S3
- Use case:
  - Restore an on-premises MySQL database into RDS
- Steps:
  1. Take a backup of the on-premises database
  2. Upload backup file to Amazon S3
  3. Restore the backup from S3 into a new RDS MySQL instance

## Restoring Aurora MySQL from Amazon S3
- Requires a specific backup method
- Steps:
  1. Take an on-premises database backup using **Percona XtraBackup**
  2. Upload the backup file to Amazon S3
  3. Restore the backup into a new Aurora MySQL cluster
- Key difference:
  - RDS MySQL: Standard database backup
  - Aurora MySQL: **Percona XtraBackup is required**

## Aurora Database Cloning
- Creates a new Aurora cluster from an existing one
- Common use case:
  - Clone a production database to create a staging or test environment
- Benefits:
  - Extremely fast
  - More efficient than snapshot and restore
  - No impact on production workloads

## How Aurora Cloning Works (Copy-on-Write)
- Initial state:
  - Clone shares the same underlying data volume as the source database
  - No data is copied initially
- As changes occur:
  - Modified data is copied to new storage blocks
  - Original and cloned databases diverge only where changes exist
- Advantages:
  - Fast creation
  - Cost-efficient storage usage
  - Ideal for testing, staging, and experimentation

## Key Takeaways
- Automated backups:
  - RDS: Optional
  - Aurora: Mandatory
- Manual snapshots:
  - Long-term retention
  - Useful for cost optimization
- Restore operations:
  - Always create new databases
- Aurora cloning:
  - Fast, efficient alternative to snapshot and restore
  - Perfect for staging environments

---

# 9. RDS and Aurora Security

## Encryption at Rest
- Data is encrypted on disk volumes for both RDS and Aurora
- Uses AWS Key Management Service (KMS)
- Applies to:
  - Primary (master) database
  - All read replicas
- Must be enabled **at database creation time**
- Important rules:
  - If the primary database is **not encrypted**, read replicas **cannot be encrypted**
  - Encryption settings cannot be changed in-place

## Encrypting an Existing Unencrypted Database
- Direct encryption of an existing database is **not possible**
- Required process:
  1. Take a snapshot of the unencrypted database
  2. Restore the snapshot as a **new encrypted database**
- This snapshot-and-restore approach is the only supported method

## Encryption in Transit (In-Flight Encryption)
- Encrypts data between clients and the database
- Enabled by default for RDS and Aurora
- Uses TLS (SSL) encryption
- Client requirements:
  - Must use AWS-provided TLS root certificates
  - Certificates are available from the AWS documentation

## Database Authentication
- Supported authentication methods:
  - Traditional username and password
  - IAM-based authentication
- IAM Database Authentication:
  - Allows AWS resources (e.g., EC2 with IAM roles) to authenticate without passwords
  - Centralized access control using IAM
  - Reduces credential management overhead

## Network Security
- Access is controlled using **Security Groups**
- Security Groups allow:
  - Restricting inbound traffic by port
  - Allowing or denying specific IP addresses
  - Allowing access from specific security groups
- Provides network-level isolation for databases

## Administrative Access
- RDS and Aurora are fully managed services
- SSH access to database instances is **not available**
- Exception:
  - SSH access is possible only with **RDS Custom** (special use case)

## Audit Logging
- Audit Logs track:
  - Database activity
  - Queries and operational events
- Logs are retained for a limited time by default
- Long-term retention:
  - Export Audit Logs to **Amazon CloudWatch Logs**
  - Enables monitoring, alerting, and historical analysis

## Key Takeaways
- Encryption:
  - At rest via KMS
  - In transit via TLS
- IAM authentication improves security and credential management
- Network access is tightly controlled using security groups
- No SSH access for standard RDS and Aurora
- Audit Logs should be sent to CloudWatch Logs for long-term visibility

---

# 10. Amazon RDS Proxy

## Overview
- Amazon RDS Proxy is a **fully managed, serverless database proxy** deployed within your VPC
- Works with both **RDS and Aurora**
- Applications connect to the proxy instead of directly connecting to the database

## Why Use RDS Proxy
- Applications typically open many database connections
- Direct connections can:
  - Exhaust database CPU and memory
  - Cause connection storms
  - Lead to timeouts and instability
- RDS Proxy **pools and shares database connections**
  - Many application connections → fewer database connections
- Improves overall database efficiency and stability

## Connection Pooling and Efficiency
- Applications connect to the proxy
- Proxy maintains a pool of connections to the database
- Benefits:
  - Reduced number of open database connections
  - Lower CPU and RAM usage on the database
  - Fewer connection timeouts
- Exam keyword:
  - “Too many connections” → **RDS Proxy**

## Serverless and High Availability
- Fully serverless:
  - No capacity planning required
  - Automatically scales based on demand
- Highly available:
  - Deployed across multiple Availability Zones
- No operational overhead for managing the proxy

## Failover Improvements
- RDS Proxy significantly improves failover handling
- During database failover:
  - Applications stay connected to the proxy
  - Proxy transparently redirects connections
- Failover time reduction:
  - Up to **66% faster**
- Supported for:
  - RDS
  - Aurora
- Exam keyword:
  - “Reduce failover time” → **RDS Proxy**

## Supported Databases
- MySQL
- PostgreSQL
- MariaDB
- Microsoft SQL Server
- Aurora MySQL
- Aurora PostgreSQL

## Application Integration
- No application code changes required
- Only change:
  - Update the database endpoint to the **RDS Proxy endpoint**
- Applications remain unaware of failovers and scaling events

## IAM Authentication and Secrets Management
- RDS Proxy can enforce **IAM-based authentication**
- Benefits:
  - No hard-coded database credentials
  - Centralized access control via IAM
- Credentials are securely stored in:
  - **AWS Secrets Manager**
- Exam keyword:
  - “Enforce IAM authentication for RDS” → **RDS Proxy**

## Network Security
- RDS Proxy is **never publicly accessible**
- Accessible only from within the VPC
- Enhances security by eliminating internet exposure

## Lambda and RDS Proxy (Critical Use Case)
- AWS Lambda functions:
  - Scale rapidly
  - Start and stop frequently
  - Can create thousands of short-lived connections
- Problem:
  - Direct Lambda → RDS connections cause connection storms
- Solution:
  - Lambda connects to RDS Proxy
  - Proxy pools and manages connections efficiently
- Result:
  - Lambda overloads the proxy (by design)
  - Database remains stable
- Exam keyword:
  - “Lambda + RDS connection issue” → **RDS Proxy**

## Key Takeaways
- RDS Proxy:
  - Pools and manages database connections
  - Reduces CPU, memory usage, and timeouts
  - Improves failover time by up to 66%
  - Enforces IAM authentication
  - Stores credentials in Secrets Manager
  - Is serverless, highly available, and VPC-only
- Ideal for:
  - High-connection workloads
  - Lambda-based architectures
  - Secure, resilient database access

---
