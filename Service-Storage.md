## EBS

1. **Block Storage**
  
Amazon EBS is a block-level storage service for EC2 instances that provides a virtual hard disk in the cloud.
You can create a volume of fixed size, attach it to an EC2 instance, mount it, detach and reattach it anytime without losing data.
Internally, AWS stores data in small blocks for high speed and reliability, but to the user it appears as a single disk drive.
It is mainly used for operating systems, databases, and applications that need fast and persistent storage.
Your laptop hard disk.
You don’t see blocks.
You just see C Drive / D Drive.
Same in AWS — EC2 sees one disk, but AWS internally manages blocks.

2. **Amazon EBS**

Amazon EBS = Virtual Hard Disk for EC2
You create a disk (20GB, 50GB, 100GB etc.)
Attach to EC2
Format it (ext4, XFS etc.)
Mount it (/mnt, /data, etc.)
Use it like normal hard drive.
So EBS = Cloud Hard Disk.

Stop EC2, Detach EBS, Attach to another EC2, Data stays safe. This is huge benefit because: Instance can die and Data still safe in EBS

3. **Multiple attach**

Normally:
1 EBS → 1 EC2
But some volume types allow:
1 EBS → Multiple EC2

Warning:
If two servers write same file → corruption possible. So apps must handle it carefully.

4. **Availability Zone**

EBS exists in one AZ only.
Example: EC2 in ap-south-1a and EBS must also be ap-south-1a

Because EBS is physically stored in that AZ.
Important: Device failure → safe
            Entire AZ failure → data lost
            So we use Snapshots.

5. **Snapshot Backup**

Snapshot = Photo of Disk Stored in S3.
Use cases: To Move data to another AZ
           Move data to another Region
Backup Flow:
You take EBS Snapshot(create snapshot)
AWS stores snapshot in S3 internally
You copy snapshot to another Region (S3 makes this easy)
You create new EBS from snapshot
Attach to EC2

6. **Cross region copy**

To migrate EBS data across regions, we create a snapshot, copy it to the target region, and then create a new EBS volume from that snapshot and attach it to an EC2 instance

You have EBS in Mumbai region
You want same data in US region

Flow

Take Snapshot of EBS in Mumbai
AWS stores snapshot internally in S3 (Mumbai region)
Copy Snapshot to US Region
AWS moves snapshot data through its internal S3 system
In US Region → Create new EBS Volume from Snapshot
Attach this new EBS to EC2 in US

You do not manually open S3, You do not upload/download files Everything is handled by AWS internally
You only click Create Snapshot → Copy Snapshot → Create Volume

7. **AWS EBS Volume types**

EBS volumes are mainly SSD for performance (gp, io) and HDD for low-cost storage (st, sc), chosen based on speed and budget needs.

```

| Volume Type                      | Storage Type | Best For                                     | Speed / Performance                 | Cost       | Simple Interview Line                                  |
| ---------------------------------| ------------ | -------------------------------------------- | ----------------------------------- | ---------- | -------------------------------------------------------|
| gp3 / gp2 (General Purpose SSD)  | SSD          | Most applications, websites, small-medium DB | Balanced speed                      | Medium     | Default EBS. Good performance + good price.            |
| io1 / io2 (Provisioned IOPS SSD) | SSD          | High-performance DB, banking, critical apps  | Very High Speed & IOPS              | High       | Used when application needs very fast disk operations. |
| st1 (Throughput Optimized HDD)   | HDD          | Big data, log processing, large files        | High throughput but slower than SSD | Low-Medium | For large data where speed is not ultra-critical.      |
| sc1 (Cold HDD)                   | HDD          | Backup, archive, rarely used data            | Slow                                | Very Low   | Cheapest option for infrequent access.                 |
| Magnetic (Old Type)              | HDD          | Legacy systems only                          | Very Slow                           | Low        | Old generation, rarely used now.                       |

gp → General apps
io → Intensive performance
st → Storage heavy workloads
sc → Super cheap archive

```
## Instance Store

Instance Store is local storage that lives inside the physical server (host) where your EC2 is running. It is very fast because it is directly connected to the machine. If the EC2 instance stops or moves to another machine, all data is lost.
EBS = External Hard Drive (safe, detachable)
Instance Store = Laptop’s internal temporary disk

* **How our EC2 Instance runs**

AWS is huge physical data centers + servers + networking + storage.

AWS have physical hardware servers
On those servers AWS installs a Hypervisor (virtualization software)
Hypervisor divides one big physical server into many virtual machines
When we run EC2 instance aws allocates any free virtual machine(host) on which EC2 instance runs.

1. When Reboot EC2 
AWS keeps you on same physical server
Just restarts the OS
Instance Store data stays

2. Stop → Start EC2
AWS may:
Free that physical server
Move your EC2 to another free server
New hardware = New Host
Instance Store disk was attached to old server → Data Lost

## EFS

Amazon EFS is a fully managed, scalable network file storage service that allows multiple Linux EC2 instances to access the same file system simultaneously using NFS. 
Multiple EC2 servers can open the same storage at the same time
It is mainly used for file sharing
Works like a Google Drive / Office Shared Folder, but inside AWS

* **How it works**

You create an EFS file system inside your VPC
AWS gives it mount targets (IPs) in subnets (Mount Target = Connection point + IP address)
Best practice: create mount target in multiple AZs If one AZ fails → another still works → High Availability
Your Linux EC2 instances mount it like a normal folder
Example: /mnt/efs
Now all EC2s can read / write / edit / delete files
So EFS is remote shared storage, not attached like EBS.

Supports NFS protocol → means network file sharing
Only for Linux (Windows not supported)
Data is automatically scalable — no need to increase size manually
Fully managed by AWS (no server maintenance)

* **Performance Modes(Speed control)**

1. General Purpose - Best for websites, CMS, normal apps it has Low latency, fast response
2. Elastic Throughput - Automatically adjusts speed based on usage
3. Max I/O - For huge workloads & many operations it has Slightly higher latency
4. Provisioned Throughput - You manually set fixed speed
5. Bursting mode - Speed increases temporarily when needed

* **How to use EFS on EC2

Install EFS utilities on Linux EC2 (sudo dnf -y install amazon-efs-utils)
Run mount command  (sudo mount .efs efs:id /directory) (id=file syatem id, directory=where we want to mount)
Folder appears like normal disk
Start using it like /mnt/data

## FSx

Amazon FSx is a fully managed file storage service in AWS.
It provides high-performance shared file systems where AWS manages servers, patches, backups, scaling, and maintenance, so we don’t have to manage file servers manually.
It is different from EFS because FSx supports both Windows and Linux environments and offers advanced enterprise file system features.

* **FSx Flavours**

```

| FSx Type                    | Best For                   | Supported OS          | Protocols       | Key Features                                           | Example Use Case                                     |
| ----------------------------| -------------------------- | --------------------- | --------------- | ------------------------------------------------------ | ---------------------------------------------------|
| FSx for Windows File Server | Windows Applications       | Windows               | SMB             | Active Directory integration, quotas, deduplication    | Company shared drive, .NET apps, Windows file server |
| FSx for Lustre              | High-Performance Computing | Linux                 | Lustre          | Very low latency, very high throughput, S3 integration | Machine Learning, Big Data, Scientific research      |
| FSx for NetApp ONTAP        | Enterprise Storage         | Windows, Linux, macOS | NFS, SMB, iSCSI | Snapshots, cloning, replication, scalable              | Large corporate storage, ERP systems                 |
| FSx for OpenZFS             | Advanced File Management   | Linux, Windows, macOS | NFS             | Compression, snapshots, cloning, cost-effective        | Dev/Test environments, analytics workloads           |

Windows → Office / Company Files
Lustre → Speed / Supercomputing
NetApp → Enterprise / Big Companies
OpenZFS → Flexible / Cost-Effective Advanced Features

FSx provides different managed file systems based on workload — Windows for Microsoft apps, Lustre for high-performance computing, NetApp for enterprise storage, and OpenZFS for advanced flexible storage.

```

* **Deployment Options**

Windows, NetApp, OpenZFS → Single AZ or Multi-AZ
Lustre → Only Single AZ

## S3 Overview

Amazon S3 is a highly scalable and secure object storage service used to store unlimited files like logs, media, and backups. It organizes data into buckets and objects using a flat structure. It integrates with EC2, Lambda, and IAM, and provides high durability by replicating data across multiple availability zones. It is commonly used for static websites, media storage, and application logs, with a 5 TB max file size and globally unique bucket names.

Each file stored in S3 is an Object. Which contains-
Key – file name
Value – file data
Metadata – extra info
Version ID (if versioning enabled)

S3 works closely with:
EC2 – store backups, logs, media
Lambda – trigger functions when file uploaded
IAM – control who can access files
We can say S3 integrates with EC2, Lambda, and IAM to create secure and automated cloud architectures.

S3 can be access and manage using AWS Console (GUI), AWS CLI, SDK (Programming languages), REST API both developers and non-developers can use it easily.

* **File Structure**

S3 does not use real folders because it is designed for massive scalability and performance. Instead of maintaining a complex directory tree, S3 keeps all files in one flat structure and uses parts of the file name to make it look like folders in the console.
example - music/song1.mp3
There is no actual folder, only object names with “/”.

S3 is commonly used for storage of Application Logs, Images / Videos / Audio, CI/CD Artifacts, Backups, Static Website Hosting Basically, any large file storage need.

* **How S3 Improves Web App Performance and Scalability**

Imagine you are building a website like YouTube. Your web server keeps the small files such as HTML, CSS, and JavaScript, but videos and images are very large in size. If you store these heavy media files on the same server, the website becomes slow, costly, and hard to scale.
So instead, you store videos and images in an S3 bucket and put their S3 links (URLs) inside your HTML pages. When a user opens the website, the web server sends the page, but the media is downloaded directly from S3. This reduces server load, improves speed, and makes the application cheaper and highly scalable.

* **Durability and Availability**

When you upload a file:
AWS automatically replicates it across multiple Availability Zones.
Even if one data center fails → data safe.
Durability is 11 nines (99.999999999%) – very high reliability.

* **Unique Bucket Name Rule**
  
Because bucket name becomes part of URL: example https://bucketname.s3.amazonaws.com/file.jpg

This link is the public URL of an S3 bucket or file. AWS automatically creates this format because S3 is accessed over the internet using HTTP/HTTPS. The bucket name becomes part of the URL so AWS can uniquely identify which bucket the request belongs to. It is used in websites, applications, and APIs to upload, download, or share files stored in S3. That’s why the bucket name must be globally unique — so there is no confusion about whose data should be opened.

* **S3 Limits/Restrictions**

```
| Feature             | Limit                      |
| ------------------- | -------------------------- |
| Objects             | Unlimited                  |
| Max File Size       | 5 TB per object            |
| Buckets per Account | 100 (can increase to 1000) |

```

## S3 Storage Classes

S3 Storage Classes are different cost and performance options in Amazon S3. We choose a storage class based on how frequently we access data, how fast we need it, and how much we want to spend.

```

| Storage Class        | Access Speed  | Cost     | Availability Zones | Best Use Case             | Important Notes               |
| ---------------------| ------------- | -------- | ------------------ | ------------------------- | ----------------------------- |
| S3 Standard          | Milliseconds  | High     | Multi-AZ           | Active websites, apps     | 11 nines durability           |
| S3 Standard-IA       | Milliseconds  | Medium   | Multi-AZ           | Backups, infrequent files | Retrieval fee, 30-day minimum |
| S3 One Zone-IA       | Milliseconds  | Low      | Single AZ          | Re-creatable data         | Risk of AZ failure            |
| Glacier Instant      | Milliseconds  | Low      | Multi-AZ           | Rare access but need fast | 90-day minimum                |
| Glacier Flexible     | Minutes–Hours | Very Low | Multi-AZ           | Archives, old logs        | Retrieval delay               |
| Glacier Deep Archive | 12–48 Hours   | Cheapest | Multi-AZ           | Compliance, legal data    | 180-day minimum               |
| Intelligent-Tiering  | Auto          | Varies   | Multi-AZ           | Unknown usage pattern     | Small monitoring fee          |

Need instant access & frequent use → S3 Standard
Instant but rare use → Standard-IA
Cheapest but can risk single AZ → One Zone-IA
Archive with fast retrieval → Glacier Instant
Archive with delay acceptable → Glacier Flexible
Very old data, cheapest → Glacier Deep Archive
Don’t know usage pattern → Intelligent-Tiering

S3 Storage Classes help balance cost, speed, and durability — Standard for active data, IA for less frequent access, Glacier for archives, and Intelligent-Tiering when access pattern is unknown.

![](https://github.com/Shravani512/AWS-Solutions-Architect-Associate-Certification/blob/ea726d23e940a87da044013a2b976491f56925a4/Images/Storage_classes.png)
