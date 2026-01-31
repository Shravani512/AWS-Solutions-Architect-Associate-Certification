## EBS Demo Part 1

In this demo we create an EBS volume, attach it to an EC2 instance, format and mount it, store data, then detach and reattach it to another instance to show that EBS provides persistent, movable block
storage within the same Availability Zone.

## EBS Demo Part 2

In this demo we learn how to move EBS data between different Availability Zones and Regions. Since an EBS volume is locked to one AZ, we cannot attach it directly to another AZ, so we create a snapshot and
then create a new volume from that snapshot in the target AZ/Region. This shows how AWS ensures data portability while maintaining AZ isolation.

## Instance Store Demo

In this demo, we are learning how EC2 Instance Store storage works and why it is temporary. We launch an EC2 instance that supports instance store, create a filesystem on the instance store disk, mount it,
and store some test data to observe its behavior.
Then we reboot the instance to see that data remains, but when we stop and start the instance, the data disappears because the instance moves to a new physical host. This demonstrates that Instance Store
is ephemeral storage meant only for temporary data like cache or scratch files, not permanent storage.

## EFS Demo

In this demo, we are learning how to create and use Amazon EFS (Elastic File System) as shared storage between multiple EC2 instances. We create one EFS file system in a VPC, configure network access and
mount targets, and then mount the same EFS on two different EC2 servers.
After mounting, we create files on one server and verify that the same files instantly appear on the other server, proving that EFS provides simultaneous read/write shared storage across instances and
availability zones. This shows that EFS is used for centralized, scalable, and persistent shared storage in cloud environments.

##  S3 Demo

In this demo we are learning the basic operations of Amazon S3 (Simple Storage Service) — how to create a bucket, upload files, organize them using folders (prefixes), manage permissions, and finally delete
objects and the bucket. It demonstrates how S3 stores data as objects inside bucketsand how security is private by default.
We also understand practical actions like uploading files, moving objects, simulating folders, handling access permissions, and cleaning up resources, which shows how S3 is used as scalable, durable object
storage for images, videos, backups, and static website content in real-world cloud applications.

## S3 Storage Class Demo

In this demo we are learning how to choose and change S3 Storage Classes to optimize cost vs access speed for files. While uploading an object, we select a storage class like Standard, One-Zone IA, or Glacier
depending on how frequently the file will be used.
We also learn that storage class is not permanent — we can later edit an object’s properties and move it to another class. This shows how S3 helps manage data economically by storing frequently used data
in faster storage and rarely used data in cheaper storage without deleting or re-uploading files.

## S3 Versioning Demo

In this demo we are understanding how S3 Versioning protects data from accidental delete and overwrite. First we observe normal behavior with versioning disabled where files are permanently deleted or replaced.
Then we enable versioning and see that every upload creates a new version ID, so old files remain safe and recoverable.
We also learn about delete markers, suspending versioning, and MFA delete, which show how S3 manages file history and adds extra security. Overall, the demo teaches how versioning helps in data backup,
recovery, and safe file management in S3.

## S3 ACL and Resource Policies Demo

In this demo we are learning how to control access to an S3 bucket using Resource (Bucket) Policies instead of only IAM policies. We create a bucket with one user and then test how different users—same account, different account, and public users—can or cannot access files based on permissions.
We then apply folder-level permissions (GET, DELETE, LIST) through bucket policies to allow specific actions only on selected folders like logs, traces, or media. Overall, the demo shows how S3 Resource Policies help in fine-grained access control, cross-account sharing, and secure public access management.

## S3 Static website hosting

In this demo we are hosting a static website using Amazon S3 instead of a traditional web server. We upload HTML, CSS, images, and a custom 404 page into an S3 bucket and enable Static Website Hosting so S3 can serve these files like a website.
Then we configure public read permissions using bucket policy and public access settings, allowing anyone on the internet to open the site through the S3 website endpoint. Overall, this shows how S3 can act as a low-cost, scalable hosting solution for static websites without needing EC2 or servers.

## S3 Pres Signed URLs Demo

In this demo we are sharing a private S3 file securely using a Pre-Signed URL instead of making the bucket public. The bucket remains private, but we generate a temporary link that allows anyone with the URL to access the file only for a limited time.
It also shows that the URL works only with the permissions of the user who created it. If the user does not have GetObject permission, the pre-signed URL will also fail. This demonstrates controlled, time-based and secure file sharing in S3 without exposing data permanently.

## S3 Access Points Demo
In this demo we are using S3 Access Points to give different users separate and controlled entry points to the same bucket instead of managing one big bucket policy. Each access point acts like a custom gateway with its own permissions for a specific team (developers, finance, etc.).
This allows fine-grained access control without changing the main bucket settings again and again. The bucket stores data centrally, while access points handle who can read, upload, or modify objects, making large-scale permission management easier and more secure.