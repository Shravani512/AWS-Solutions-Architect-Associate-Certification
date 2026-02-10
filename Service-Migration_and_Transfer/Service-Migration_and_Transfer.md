## Migration/Transfer agenda and Introduction

**AWS Cloud Migration Process**

1. Assessment and Inventory:
Catalogue your digital assets, including applications, databases, storage, and dependencies present in your current environment.
2. Categorization:
Group applications based on complexity, interdependencies, and business criticality to streamline migration planning.
3. Determining Cloud Services:
Choose the most suitable AWS services based on your assessment to ensure compatibility and optimal performance in the cloud.
4. Migration Planning:
Develop detailed migration strategies for each application. Decide whether to re-host, refactor, re-architect, or rebuild your applications.
5. Migration Execution:
Execute the migration plan while monitoring progress and ensuring a smooth transition to AWS.

**AWS Migration Services** (Names ONLY)**

1. AWS Migration Hub
2. Application Discovery Service
3. Application Migration Service (MGN)
4. Database Migration Service (DMS)
5. Elastic Disaster Recovery
6. Mainframe Modernization
7. AWS DataSync
8. AWS Transfer Family
9. AWS Snow Family

## Migration Hub

AWS Migration Hub is a central dashboard that helps you plan, track, and monitor the migration of applications and servers from on-premises or other clouds to AWS.

**1. Discovery Phase**
Migration Hub helps you collect details about your existing servers and apps using:
1. Agent-based discovery
Install an agent on servers
Collects detailed metrics:
CPU usage
Memory usage
Network traffic

2. Agentless discovery
No software installation
Collects basic system information

**2.Assessment Phase**

After discovery, Migration Hub analyzes the collected data and helps you decide:
Which EC2 instance type to us
Estimated cost after migration
Whether workloads are right-sized

**3. Migration Phase**
Once planning is done, Migration Hub does NOT migrate by itself.
Instead, it:
Integrates with migration tools
Tracks migration progress in one place

Common integrated services:
AWS Application Migration Service
AWS Database Migration Service (DMS)
Partner migration tools

Discover → Assess → Migrate → Track

## AWS Application Discovery Service

AWS Application Discovery Service is a service that helps you understand your existing on-premises environment before migrating to AWS. It automatically collects information about your servers, applications, and how they communicate with each other. This gives you a complete and accurate picture of what you are planning to migrate.

**Why is this service needed**

In real environments, applications are not isolated. A single app may depend on:
Multiple servers, Databases, Background processes, Network connections to other apps
Most organizations do not have updated documentation showing these dependencies. If you migrate blindly, applications may break because a hidden dependency was missed.

Application Discovery Service is needed to remove guesswork from migration planning and prevent downtime caused by missing dependencies.

**How does Application Discovery Service solve this**

It works by collecting real data from your environment using two methods:

**- Agent-based discovery**
Lightweight agents are installed on servers to collect deep details like running processes, network traffic, and performance metrics.

**- Agentless discovery**
A collector VM is deployed in VMware environments to gather basic system and utilization data without installing agents on each server.

This data is securely sent to AWS and stored in Amazon S3, where it can be analyzed using Migration Hub and Athena.

## Application Migration Service

AWS Application Migration Service (MGN) is the service that actually moves your servers and applications to AWS. After you understand what you have (using Application Discovery Service) and plan the migration (using Migration Hub), Application Migration Service performs the real lift-and-shift migration of physical servers, virtual machines, and cloud servers into AWS.

In simple words:

Discovery tells you what to move, Migration Hub tracks it, and Application Migration Service moves it.

**Why is this service needed?**

Manually migrating servers is slow, risky, and error-prone. You would have to:

Copy data manually
Recreate disks and configurations
Stop applications for long periods
Risk data inconsistency

Application Migration Service is needed to automate this entire process, reduce downtime, and make large-scale migrations practical and repeatable.

**How does it solve the problem**
Application Migration Service works by continuously replicating data from your source servers into AWS.

First, a replication agent is installed on each source server. This agent continuously copies disk data to AWS into a staging area, where equivalent EBS volumes are created. If your on-prem server has a 10-GB disk, AWS creates a 10-GB EBS volume and keeps it in sync.

Because replication is continuous, AWS always has the latest version of your data, which allows testing and cutover without stopping the source server.

**Migration, Testing, and Cutover** 

While replication is running, you can launch test EC2 instances in a separate migrated resources subnet to verify that applications work correctly in AWS. When you are confident, you perform a cutover, and the service launches production EC2 instances using the latest replicated data.

Even after cutover, replication continues. This means if something goes wrong, you can retry or re-migrate using updated data, making the process safer and reversible.

**Real-World Scenario** (Easy to Visualize)**

A company has an on-premises ERP system running on VMware with multiple disks. They want to migrate it to AWS with minimal downtime.

They:
1. Use Application Discovery Service to understand dependencies
2. Track progress using Migration Hub
3. Install replication agents and migrate servers using Application Migration Service

The data keeps syncing in the background, they test the app in AWS, and finally cut over during a short maintenance window. The ERP system goes live on AWS with minimal downtime and no data loss.

## Database Migration Service

AWS Database Migration Service (DMS) is a managed service that helps you move databases to AWS safely, with minimal downtime and no data loss. If Application Migration Service moves servers and applications, DMS is focused only on databases.

In simple words:
DMS keeps your source and target databases in sync until you are ready to switch.

**What problem does it solve?**

Without DMS:
- You would need long downtime to copy database data
- Data could become inconsistent
- Migrating between different database engines would be risky
- Manual scripts would be complex and error-prone

DMS solves the problem of migrating live databases without interrupting business operations.

**How does DMS work?**

The migration starts with understanding the database environment. Tools like DMS Fleet Advisor collect information from your existing databases—such as engine type, size, and usage—to help assess migration readiness.

If the migration involves different database engines (for example, Oracle to PostgreSQL), the Schema Conversion Tool (SCT) analyzes and converts schemas, tables, and database objects so they work correctly in the target database.

Once planning is complete, AWS creates a replication instance, which acts as the engine that moves data. You define source and target endpoints, telling DMS where the data comes from and where it should go.

- Continuous Replication 

DMS copies the initial data from the source database to the target. After that, it uses Change Data Capture (CDC) to continuously track and apply changes made on the source database to the target.

Because of this:
Source and target stay in sync
You can test applications on the target
Final cutover requires only a short downtime

**Migration Types**

1. Full Load – One-time copy of all data
2. Full Load + CDC – Copy data, then keep syncing changes
3. CDC Only – Only replicate ongoing changes

This flexibility allows DMS to fit both simple and complex migration needs.

**Real-World Scenario**

A company has an on-prem Oracle database used by a live application. They want to move to Amazon Aurora PostgreSQL.

They:
- Analyze the database using Fleet Advisor
- Convert schema using Schema Conversion Tool
- Use DMS Full Load + CDC to migrate data
- Test the application on Aurora
- Perform final cutover with minimal downtime
- Users barely notice the migration.

AWS DMS migrates live databases to AWS using continuous replication and CDC, ensuring minimal downtime and zero data loss.

## AWS Mainframe Modernization

AWS Mainframe Modernization is a service that helps organizations move and modernize legacy mainframe applications (usually running on very old systems) into AWS-managed cloud environments. It doesn’t just migrate the application—it also updates how the application runs, so it can scale, heal, and integrate like a modern cloud application.

In simple terms:
It helps old mainframe apps live comfortably in the cloud.

**Why is this service needed?**

Many large enterprises (banks, insurance, government) still run business-critical applications on mainframes written in COBOL or PL/I. These systems:

- Are expensive to maintain
- Are hard to scale
- Depend on aging hardware and skills
- Don’t integrate easily with modern cloud services

AWS Mainframe Modernization is needed to reduce cost, remove hardware dependency, and modernize safely without breaking critical systems.

**1. Refactoring (Modernization path)**

Refactoring is chosen when you want to modernize the application itself. AWS tools (especially AWS Blueprints) automatically analyze and transform legacy code into cloud-ready architectures.

This includes:
- Converting legacy cod
- Updating data formats
- Breaking monoliths into services
- Adding testing and CI/CD pipelines

This path takes more effort but gives long-term flexibility and cloud-native benefits.

**2. Replatforming (Fast migration path)**

Replatforming focuses on speed. The application is moved to AWS without changing source code. The runtime environment changes, but the application logic stays the same.

This is ideal when:
You need to exit mainframe hardware quickly
You want minimal risk
Modernization can be done later

**Real-World Scenario (Easy Example)**

A bank runs its core transaction system on a mainframe written in COBOL. Maintaining the system is costly and slow.
- Using AWS Mainframe Modernization, the bank:
- Assesses the application
- Replatformes first to move quickly to AWS
- Later refactors parts of the system for microservices
- Uses CI/CD to deploy updates safely

The system becomes cheaper, scalable, and future-ready—without disrupting customers.

**AWS Mainframe Modernization helps migrate and modernize legacy mainframe applications to AWS using refactoring or replatforming with managed runtimes and CI/CD support.**

## AWS Transfer Family

Think of AWS Transfer Family as AWS giving you a fully managed, secure FTP server without the headache of building or maintaining one. It allows organizations to transfer files into and out of AWS storage using familiar file transfer protocols while AWS manages all the infrastructure behind the scenes.

It supports Amazon S3 and Amazon EFS as storage targets and works with standard protocols like SFTP, FTPS, FTP, and AS2, making it easy to migrate or integrate legacy systems with AWS

**How it’s different from DataSync**

While DataSync is mainly used for bulk data migration, AWS Transfer Family is designed for ongoing file exchange. It behaves like a traditional FTP server but is highly available, scalable, and secure by default, with no servers for you to manage.

**How it works**

You create a Transfer Family server, and AWS provides a public endpoint so external users or partners can upload or download files. If access needs to be restricted, the server can be placed inside a VPC using a VPC endpoint.

When users transfer files, they are directly stored in S3 or EFS. AWS also integrates with CloudWatch, giving visibility into transfers, errors, and activity logs.

**SFTP Connector Integration**

AWS Transfer Family also includes an SFTP Connector, which allows AWS to securely pull or push files from external SFTP servers (on-premises or cloud). This helps merge external data with AWS-based analytics, reporting, or data lake workloads without manual intervention.

**Conclusion**

AWS Transfer Family simplifies secure file transfer by providing a fully managed FTP/SFTP/FTPS/AS2 service that connects directly to S3 and EFS. It’s ideal for businesses that rely on continuous file exchange and want AWS to handle availability, security, and scalability.

**AWS DataSync**

AWS DataSync is a managed data transfer service that helps organizations move large amounts of data quickly and securely between on-premises storage and AWS storage services. Instead of building custom scripts or managing complex tools, DataSync automates the entire migration and synchronization process.

It is commonly used when companies want to migrate, replicate, or continuously sync data to AWS services like Amazon S3, EFS, or FSx.

**How DataSync Works**

DataSync works using four simple building blocks. First, a DataSync Agent is deployed close to the data source (on-premises or cloud). This agent reads data from the storage system and communicates with AWS DataSync.

Next, you define Locations, which represent the source and destination of the data—such as an on-premises NFS server and an S3 bucket. Then, you create a Task, which acts like a blueprint describing how the data should be transferred. Every time you run this blueprint, it becomes a Task Execution, showing real-time progress and results.

**Task Execution Lifecycle**

When a task runs, it moves through clear stages: it may wait in queued state, then launch, prepare by identifying which files need copying, transfer the data, and finally verify the integrity of the copied files. This structured flow makes the transfer reliable and easy to monitor.

**Key Capabilities**

DataSync is optimized for high-speed transfers, supports NFS, SMB, and S3, performs incremental copies, encrypts data during transfer, and provides detailed CloudWatch monitoring and logs. It also supports automation and scheduling, making it suitable for recurring data sync jobs.

**Common Use Cases**

Organizations use AWS DataSync to migrate data from on-premises environments to AWS, synchronize data across AWS regions for disaster recovery, or move data from other cloud providers like Azure or GCP into AWS.