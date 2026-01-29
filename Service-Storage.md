### EBS

* **Block Storage**
  
Amazon EBS is a block-level storage service for EC2 instances that provides a virtual hard disk in the cloud.
You can create a volume of fixed size, attach it to an EC2 instance, mount it, detach and reattach it anytime without losing data.
Internally, AWS stores data in small blocks for high speed and reliability, but to the user it appears as a single disk drive.
It is mainly used for operating systems, databases, and applications that need fast and persistent storage.
Your laptop hard disk.
You don’t see blocks.
You just see C Drive / D Drive.
Same in AWS — EC2 sees one disk, but AWS internally manages blocks.

* **Amazon EBS**

Amazon EBS = Virtual Hard Disk for EC2
You create a disk (20GB, 50GB, 100GB etc.)
Attach to EC2
Format it (ext4, XFS etc.)
Mount it (/mnt, /data, etc.)
Use it like normal hard drive.
So EBS = Cloud Hard Disk.

Stop EC2, Detach EBS, Attach to another EC2, Data stays safe. This is huge benefit because: Instance can die and Data still safe in EBS

* **Multiple attach**

Normally:
1 EBS → 1 EC2
But some volume types allow:
1 EBS → Multiple EC2

Warning:
If two servers write same file → corruption possible. So apps must handle it carefully.

* **Availability Zone**

EBS exists in one AZ only.
Example: EC2 in ap-south-1a and EBS must also be ap-south-1a

Because EBS is physically stored in that AZ.
Important: Device failure → safe
            Entire AZ failure → data lost
            So we use Snapshots.

* **Snapshot Backup**

Snapshot = Photo of Disk Stored in S3.
Use cases: To Move data to another AZ
           Move data to another Region
Backup Flow:
You take EBS Snapshot(create snapshot)
AWS stores snapshot in S3 internally
You copy snapshot to another Region (S3 makes this easy)
You create new EBS from snapshot
Attach to EC2

* **Cross region copy**

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

* **AWS EBS Volume types**

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

EBS volumes are mainly SSD for performance (gp, io) and HDD for low-cost storage (st, sc), chosen based on speed and budget needs.

```
