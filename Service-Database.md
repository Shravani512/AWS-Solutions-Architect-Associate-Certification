## Database Agenda and Introduction

**AWS database types:**
Amazon RDS
Amazon Aurora
Amazon DynamoDB
Amazon Redshift
Amazon ElastiCache
Amazon MemoryDB
Amazon Neptune
Amazon Keyspaces
Amazon Timestream
Amazon DocumentDB
Amazon QLDB

# AWS RDS 

**1. AWS RDS**
Amazon Relational Database Service (RDS) is a fully managed AWS service that allows you to run relational databases without managing servers or operating systems. AWS handles routine tasks such as backups, patching, monitoring, and failure recovery. This reduces operational complexity and helps teams focus on application development. RDS is designed for reliability, scalability, and security. It is commonly used for web, mobile, and enterprise applications.

**2. Why RDS is Needed**
Managing databases manually requires constant attention to availability, performance, and security. Tasks like applying patches, taking backups, configuring replication, and planning disaster recovery are complex and error-prone. Missing backups or delayed updates can lead to data loss or security breaches. RDS automates these responsibilities, providing a safer and more consistent database environment.

**3. Supported Database Engines**
AWS RDS supports multiple relational database engines such as MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, and Amazon Aurora. This flexibility allows organizations to migrate existing databases to AWS with minimal changes. Each engine retains its native features while benefiting from AWS-managed operations. Users can select the engine that best fits their application needs.

**4.RDS Instance Types**
RDS instance types define the compute and memory capacity of the database. General Purpose instances provide a balanced mix of CPU, memory, and networking for most workloads. Memory Optimized instances are designed for applications that require large in-memory processing. Choosing the correct instance type ensures better performance and cost efficiency. Instance sizes can be changed as workload requirements evolve.

**5. Single-AZ Deployment**
In a Single-AZ deployment, the database runs in one availability zone within an AWS region. This option is cost-effective and simple to manage, making it suitable for development, testing, and non-critical workloads. However, it does not provide automatic failover. If the availability zone fails, the database becomes unavailable.

**6. Multi-AZ Deployment**
Multi-AZ deployment improves availability by maintaining a standby database in a separate availability zone. Data is automatically replicated from the primary instance to the standby. In case of failure, AWS performs an automatic failover without manual intervention. This minimizes downtime and enhances reliability. Multi-AZ deployments are recommended for production environments.

**7. How It Works in RDS (Multi-AZ Deployment)**
- Uses **2 Availability Zones**
- Primary database runs in **AZ-1**
- Standby database runs in **AZ-2**
- No cluster architecture (only primary + standby)
- Standby instance is **not used for read traffic**

**8. Read Replicas**
Read replicas are read-only copies of the primary database used to scale read-heavy workloads. They reduce the load on the primary instance by serving read requests. Replication is asynchronous, which may cause minor replication lag. Read replicas can be promoted to standalone databases if required. They are useful for performance scaling and disaster recovery.

**9. Cross-Region Read Replicas**
Cross-region read replicas are located in a different AWS region than the primary database. They reduce latency for users located far from the primary region. They also provide regional disaster recovery protection. Data is securely replicated across regions. This setup is ideal for global applications.

**10 Multi-AZ Cluster**
A Multi-AZ cluster uses multiple instances across availability zones to provide high availability and read scalability. The primary instance handles read and write operations, while read replicas distribute read traffic. A standby instance ensures rapid recovery during failures. This architecture offers faster failover compared to standard Multi-AZ deployments. It is suitable for mission-critical workloads.

 How It Works in RDS (Multi-AZ Cluster)
- Uses **multiple Availability Zones**
- Primary instance runs in one AZ
- Read replicas run in other AZs
- Standby instance runs in another AZ
- All instances together form **one logical cluster**
- Read replicas actively serve read traffic

**11. Blue-Green Deployment**
Blue-green deployment uses two environments: blue (current production) and green (updated version). Changes are applied and tested in the green environment before switching traffic. This allows upgrades with minimal downtime and reduced risk. The previous environment can be retained for rollback if needed. It is commonly used for major updates.

**12. Storage Types in RDS**
Amazon RDS supports multiple storage options to meet performance needs. General Purpose SSD offers balanced performance and cost for most workloads. Provisioned IOPS SSD provides consistent, low-latency performance for I/O-intensive applications. Magnetic storage is an older option with lower performance and is being phased out. Choosing the right storage type directly impacts database performance.

**13. DB Parameter Groups**
DB Parameter Groups control database engine settings such as memory allocation, query execution, and logging behavior. They allow customization without modifying application code. Some parameter changes require a database restart. Parameter groups help optimize performance and maintain consistent configurations. They are specific to each database engine.

**14. DB Option Groups**
DB Option Groups enable additional database features such as encryption and advanced capabilities. These options extend the default functionality of the database engine. Not all options are supported by every engine. Option groups allow controlled customization while maintaining managed service benefits.

**15. DB Subnet Groups**
DB Subnet Groups define the subnets where RDS instances can be launched within a VPC. They ensure proper network placement and isolation. Subnet groups are required for Multi-AZ deployments. They enhance security by keeping databases in private subnets. This helps control connectivity and availability.

**17. Security Groups**
Security Groups act as virtual firewalls for RDS instances. They control inbound and outbound traffic based on defined rules. Only authorized IP addresses and ports can access the database. Security groups play a critical role in preventing unauthorized access. They are essential for database network security.

**18. Backups and Snapshots**
RDS automatically performs backups and supports manual snapshots. Automated backups allow point-in-time recovery within the retention period. Manual snapshots are useful for long-term storage and migrations. Backups protect against accidental deletion and data corruption. AWS manages backup storage securely.

**19. Monitoring and Performance**
RDS integrates with Amazon CloudWatch to monitor metrics such as CPU usage, memory, and disk I/O. Performance Insights provides visibility into database load and query performance. Enhanced Monitoring offers OS-level metrics for deeper analysis. These tools help identify performance bottlenecks and maintain stability.

**20. Encryption**
RDS supports encryption at rest and in transit. Data at rest is encrypted using AWS Key Management Service (KMS). Data in transit is protected using SSL/TLS. Encryption helps meet security and compliance requirements. It ensures sensitive data remains protected.

**21. Key Benefits of AWS RDS**
AWS RDS simplifies database management by automating operational tasks. It provides scalability, high availability, and strong security. RDS supports multiple database engines and deployment models. It reduces administrative overhead while improving reliability. Overall, RDS enables efficient and secure database operations in the cloud.


## RDS Aurora and Aurora Serverless

Amazon Aurora is a relational database in AWS, which means it stores structured data (like tables for users, orders, posts, etc.) and supports SQL queries. It’s fully compatible with MySQL and PostgreSQL, so any application using these databases can switch to Aurora without major changes.

**1. Architecture**

Aurora is cloud-native, meaning it’s designed to run perfectly in the cloud instead of just copying old traditional database designs.

1. Compute and Storage are Separate:
Traditional databases tie CPU, memory, and storage together on one machine. Aurora decouples them:
Compute → The “brain” that processes queries (CPU + RAM).
Storage → The “warehouse” that holds your actual data.

2. Distributed Storage Across Multiple AZs:
Aurora stores multiple copies of each data segment in different Availability Zones (AZs). If one AZ fails, the database still works.

3. Cluster-Based:
An Aurora database runs as a cluster, not a single instance:
Primary Instance: Handles both reads and writes.
Replicas: Handle only reads (up to 15). If primary fails, a replica can take over.
Cluster Volume: Virtual storage layer shared across all instances, automatically scaling and replicating across AZs.

4. Quorum & Gossip Protocol:
Aurora uses a system that ensures data consistency even during failures. If one copy of data goes wrong, it’s automatically corrected using other copies.

Aurora does not keep data inside the database server itself.
The database server (compute) only runs queries like read and write, while the actual data is kept in a separate shared storage system managed by AWS.
The database server talks to this storage using AWS’s private, very fast internal network, instead of saving data on its own hard disk.

**2. Why Traditional Databases Had Problems**

Old-school databases combine compute + storage, and this causes several issues:

1. Long backup times: Every snapshot copies the entire database.
2. Scaling is hard: You can’t just increase storage without downtime.
3. High costs: You must over-provision resources for peak load, wasting money during low traffic.
4. Limited availability: If the server or storage fails, the database goes down.

Aurora solves traditional database problems by automatically scaling storage and compute, so you don’t need downtime or over-provisioning. It ensures high availability and fast recovery by keeping multiple copies of data across AZs and fixing failures automatically. Using read replicas and serverless pricing, Aurora delivers high read performance while keeping costs low.

**3. Deployment Options**

1. Provisioned Aurora:
You set CPU, RAM, storage upfront. Best for predictable, high-traffic workloads. Supports global databases.

2. Aurora Serverless:
Automatically adjusts capacity based on load. Best for unpredictable or intermittent workloads. Pay only for what you use.

**4. Integrations with Other AWS Services**

Aurora integrates easily with AWS ecosystem:

S3 → Backups and recovery
CloudWatch → Monitoring performance
Lambda/ECS/EKS → Connect applications easily
Secrets Manager → Securely store credentials