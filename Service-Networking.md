- VPC
- Elastic IP
- Security Groups and NACLs
  1. Stateless firewall(NACLs)
  2. Statefull firewall(Security Groups)
 
##### Stateless Firewall
A stateless firewall filters network traffic only based on rules like source/destination IP address, port number, and protocol.
It does not remember previous packets or track ongoing connections.
It is used where simple and fast filtering is needed and traffic inspection is not complex.

Real-time example:
AWS Network ACL (NACL)

Example rule:
“Allow inbound traffic on port 80 from any IP”
Each packet is checked independently, without knowing if it is part of an existing connection.

##### Statefull Firewall
A stateful firewall filters traffic based on IP address, port, protocol, and also tracks the state of active connections (sessions).
It remembers whether a packet is part of an already established connection.
It is used where continuous, secure, and reliable connections are required.

Real-time example:
AWS Security Group

When an inbound request is allowed, the response traffic is automatically allowed, even without an outbound rule.
Used in web applications, databases, and APIs.

Stateless firewall: Checks packets individually.
Stateful firewall: Checks packets with connection context.

NACL → Supports ALLOW and DENY
Security Group → Supports only ALLOW rules

#### Important
NACLs are stateless firewalls that operate at the subnet level and control inbound and outbound traffic entering or leaving the subnet. They do not filter traffic between instances within the same subnet. Security Groups are stateful firewalls attached to instances and they control traffic between instances, even within the same subnet.

NACL → Rules are evaluated in order (rule number) 
Security Group → All rules are evaluated together

###### NACL: “Rules are evaluated in order (rule number)”
What this means- 
Every NACL rule has a rule number (100, 110, 120, etc.)
AWS checks rules from the smallest number to the largest
First matching rule wins
After a match, no more rules are checked

Example: (NACL)
Rule No	Type	Source	Action
100	HTTP	0.0.0.0/0	ALLOW
110	HTTP	1.2.3.4/32	DENY

Incoming request from 1.2.3.4 on port 80:

- AWS checks rule 100 first
- It matches → ALLOW
-  Rule 110 is never checked

Even though there is a DENY rule, traffic is allowed because lower rule number matched first.

##### Security Group: “All rules are evaluated together”
What this means-
Security Groups do NOT have rule numbers

There is no order
AWS checks all rules at once
If any rule allows the traffic → traffic is allowed
If no rule allows it → traffic is denied (implicit deny)

Example :(Security Group)
Type	Port	Source
HTTP	80	0.0.0.0/0
HTTP	80	1.2.3.4/32

Incoming request from 1.2.3.4 on port 80:

- AWS checks all rules
- Finds a matching ALLOW
- Traffic is allowed

There is no concept of deny or priority here.

##### Important
Network ACLs do not filter traffic for certain AWS-managed services like DNS, DHCP, instance metadata, and time synchronization because these services are essential for basic EC2 and VPC functionality, so AWS allows this traffic by default even if NACL rules deny other traffic.

### Load Balancer

Elastic Load Balancer Architecture
![](https://github.com/Shravani512/AWS-Solutions-Architect-Associate-Certification/blob/56842feefc8d23e9b1497885c596e8eabd8af73b/Images/Elastic_Load_Balancer.png)

- Users send requests to the ELB DNS name(DNS record is created for ELB) (not directly to servers)
- ELB keeps a list of healthy backend servers (EC2, containers, IPs)
- ELB runs health checks
- ELB forwards each new request to a healthy server using a load-balancing algorithm
- If a server becomes unhealthy, ELB stops sending traffic to it

##### Types of Load Balancing
- Classical Load Balancing
- Application Load Balancing
- Network Load Balancing

###### Classical Load Balancing
Classic Load Balancer (CLB), also known as Elastic Load Balancer generation 1, distributes incoming traffic across multiple EC2 instances for basic fault tolerance. It operates at both Layer 4 (TCP/UDP) and Layer 7 (HTTP/HTTPS), making it a hybrid of NLB and ALB functionalities.
CLB routes traffic based on IP/port or simple HTTP rules, with health checks to forward only to healthy instances across Availability Zones. It auto-scales with traffic but lacks advanced Layer 7 features like path-based routing found in ALB

##### Operation
Clients connect to CLB's DNS, which balances loads using algorithms like round-robin; it supports SSL termination similar to ALB but without modern integrations like WAF or Lambda. Designed for EC2-Classic networks, it handles both connection-oriented TCP and request-oriented HTTP traffic.

Application Load Balancers (ALB) and Network Load Balancers (NLB) differ primarily in the OSI layers they operate on and how they handle traffic termination. ALB functions at Layer 7 for HTTP/HTTPS, while NLB works at Layer 4 for TCP/UDP

###### Application Load Balancing (Layer 7 Application Layer)
![](https://github.com/Shravani512/AWS-Solutions-Architect-Associate-Certification/blob/104e433aa8f19c5ddaff46b15d147330f2894198/Images/Application_Load_Balancer.png)

ALB terminates HTTP/HTTPS connections, decrypts traffic using certificates on the load balancer, and forwards unencrypted requests to targets, enabling content-based routing like path or host rules. ALB supports HTTP/HTTPS with SSL/TLS termination at the balancer, as shown in the image where certificates reside on ALB.
Best for - Web apps, microservices 

###### Network Load Balancing (Layer 4 Transport Layer)
![](https://github.com/Shravani512/AWS-Solutions-Architect-Associate-Certification/blob/83d0745d3240469785c433b0d940c3abad86c3eb/Images/Network_Load_Balancer.png)
NLB passes through encrypted TCP/UDP traffic without termination, keeping end-to-end encryption intact and preserving client source IPs for low-latency performance. NLB handles TCP/UDP/TLS passthrough, with certificates optionally on the balancer or targets, resulting in encrypted traffic to backends.
Best For - High-performance, low-latency TCP/UDP

Main Difference:
The exact main difference is Layer 7 (ALB) vs. Layer 4 (NLB) operation: ALB inspects and routes based on application content after termination, whereas NLB forwards connections based on IP/port without inspecting higher-layer data

In real systems, ALB is used for web applications where routing depends on request content, while NLB is used for high-performance TCP/UDP workloads where latency is critical.
example-
ALB: ALB reads the HTTP request (URL path) and routes it to the correct backend service based on what the user is requesting.
NLB: NLB forwards TCP/UDP traffic directly to backend servers without inspecting it, ensuring ultra-low latency and high-speed connections.  

```
Situation	Use
Website / API / Microservices	ALB
URL-based routing needed	ALB
Game / Chat / DB / Streaming	NLB
Ultra-low latency needed	NLB
Non-HTTP protocols	NLB
```

##### Cross Zone Load Balancing
Cross-Zone Load Balancing means the load balancer distributes traffic evenly across all healthy instances in all Availability Zones, not just within the Availablity zone where the request was received.
without Cross Zone Load Balancing
User request enters AZ-1
- Traffic is sent only to instances in AZ-1
- Instances in AZ-2 may stay idle
  
  And the result is One AZ overloaded Other AZ under-used
  with cross zone load balancing User request enters AZ-1 Load balancer spreads traffic across instances in AZ-1 and AZ-2

Cross-Zone Load Balancing is achieved by enabling the cross-zone setting on the load balancer, which allows the load balancer to distribute traffic to healthy targets across all Availability Zones.
  which results Even traffic distribution, Better performance, Higher availability

![](https://github.com/Shravani512/AWS-Solutions-Architect-Associate-Certification/blob/6f6f4402ae9fa7033735db61f9025c11c9fb447d/Images/cross_load_balancer.png)
We create one load balancer in a VPC, and AWS automatically places its parts in each selected Availability Zone to handle traffic.

##### ELB Architecture
![](https://github.com/Shravani512/AWS-Solutions-Architect-Associate-Certification/blob/6133635d101dc7c1d08f20ef116e6cc1a28f68d6/Images/ELB_Architecture.png)
This diagram illustrates a secure AWS VPC architecture using public and private subnets for ELB (Elastic Load Balancer). Internet users send requests to the ELB placed in public subnets, which have internet access via an Internet Gateway for inbound traffic. ELB then forwards these requests internally to application resources like EC2 instances secured in private subnets across Availability Zones. Private subnets lack direct internet access, enhancing security by isolating backend servers from public exposure. VPC routing tables handle intra-VPC traffic between public and private subnets without leaving AWS infrastructure. This multi-AZ setup ensures high availability, as ELB distributes loads across zones if one fails.
​Overall, it balances accessibility (public ELB) with protection (private backends), a standard pattern for web apps. Private resources use NAT gateways for outbound internet needs like updates.

##### Listeners and Target groups

Listener
Listens for incoming traffic on a specific port and protocol (like HTTP 80 or HTTPS 443)
Decides what to do with the request

Target Group
A group of backend servers (EC2, pods, IPs, Lambda)
Receives traffic from the load balancer

### VPC (Virtual Private Network)

VPN (Virtual Private Network) is a technology that creates a secure, encrypted connection over a public network (like the internet), allowing users or networks to access private resources safely as if they were on the same private network.

A VPN allows remote users to securely access a private subnet by creating an encrypted tunnel between the user’s device and the VPC. Since private subnets cannot be accessed directly from the internet, the user first connects to a VPN endpoint. This VPN endpoint is attached to the VPC and associated with subnets that have routing to the private subnet. Once connected, the VPN assigns the user a private IP address, making the user appear as if they are inside the VPC network. The VPC route tables and security groups then allow traffic from the VPN to reach resources in the private subnet securely.

Example: AWS Client VPN
VPN Architecture
![](https://github.com/Shravani512/AWS-Solutions-Architect-Associate-Certification/blob/71228753392174630d68caae62391d1183b3c97f/Images/VPN_Architecture.png)

```
On-prem / User
   ↓
Customer Gateway
   ↓  (VPN tunnel)
VPN Gateway (AWS)
   ↓
VPC Subnet
```

Types of VPN routing
VPN routing = rules that tell your VPN which traffic goes where.
- Static routing = manual, fixed routes
- Dynamic routing = automatic, learns routes via BGP

### Direct Connect

Architecture
![](https://github.com/Shravani512/AWS-Solutions-Architect-Associate-Certification/blob/d262a6b3b5cb90235137863c7d0c3b204e4d90e8/Images/direct_connect_architecture.png)
AWS Direct Connect Architecture allows you to establish a dedicated, private network connection between your on-premises data center and AWS. Instead of using the public Internet, traffic flows through a high-speed, low-latency link, providing more consistent and secure connectivity. In a typical setup, your on-premises router connects to a Direct Connect location via a physical connection, which then links to one or more VPCs in AWS through Virtual Interfaces (VIFs). There are usually two types of VIFs: private VIF for accessing VPC resources and public VIF for accessing AWS public services like S3. Direct Connect can be combined with VPN as a backup to ensure high availability. Overall, this architecture improves performance, security, and reliability for hybrid cloud setups.

### VPC Peering
Earlier, if you had two different VPCs, you could not connect their subnets directly. Each VPC was isolated, so resources in one VPC (like EC2 instances or databases) couldn’t talk to resources in another VPC.

That’s where VPC Peering came into the picture. It creates a private connection between two VPCs, allowing their subnets to communicate with each other using private IPs, without going over the Internet. Now, instances in one VPC can access instances or services in another VPC securely and directly.

- VPC Peering allows private communication between two VPCs using AWS private network (no internet, no VPN).
- Can be done within the same region or across different regions (inter-region peering).
- VPCs must have non-overlapping CIDR blocks.
- Traffic is encrypted by default and stays inside AWS backbone.
- You must manually update route tables in both VPCs to enable traffic flow.
- No transitive peering → VPC A ↔ VPC B and VPC B ↔ VPC C does NOT allow A ↔ C.
- Works across different AWS accounts as well.
- Security Groups & NACLs must allow traffic explicitly.

### Transit Gateway
AWS Transit Gateway is a managed, centralized routing service that acts as a hub to connect multiple VPCs and on-premises networks, enabling transitive communication between them.
Instead of creating many one-to-one VPC peering connections, each VPC connects once to the Transit Gateway, and the gateway handles all routing between VPCs and external networks.
Transit Gateways can peer with other Transit Gateways example- Enables: Inter-region connectivity and Cross-account networking

#### What Transit Gateway Does Internally

Acts like a cloud router
All attached networks send traffic to the same gateway
The gateway decides where to forward traffic
Supports transitive routing automatically
Each VPC attaches to the Transit Gateway

For attachment:
You must select one subnet per Availability Zone
These subnets host TGW network interfaces
Traffic from VPC → TGW → destination VPC

### Private Link
AWS PrivateLink is a way to access AWS services or services running in another VPC privately and securely from your own VPC. When you use PrivateLink, your data never goes to the public internet; it stays inside AWS’s private network. You don’t need an Internet Gateway, NAT Gateway, or public IPs, which makes your setup much safer. PrivateLink creates a private endpoint inside your VPC, and through this endpoint, you can reach only the specific service you need. From your side, it feels like that service is sitting inside your own VPC, even though it is actually hosted elsewhere.

Normal (Without PrivateLink)

Your EC2 is in a private subnet
It wants to access S3 / external service
To go outside the VPC, traffic must:
Go to NAT Gateway (outbound)
Or Internet Gateway (public subnet)

This means:
Your VPC is connected to the internet
Even if restricted, exposure exists

With AWS PrivateLink
AWS creates a VPC Endpoint (ENI) inside your VPC
This endpoint has a private IP
Your EC2 sends traffic to:
10.x.x.x (private IP)
Traffic:
❌ Never goes to Internet Gateway
❌ Never goes to NAT Gateway
❌ Never uses public IPs
AWS internally routes it inside their private backbone

If traffic never leaves AWS’s private network, there is nothing on the internet that can see or attack it.

### CloudFront
Amazon CloudFront is a service that makes websites and applications load faster for users all over the world. It works by storing copies of your content (like images, videos, HTML, CSS, and JavaScript) at edge locations that are close to users. When someone requests your site, CloudFront delivers the content from the nearest edge location instead of a faraway server, which reduces delay. The original source of the content is called the origin, such as an S3 bucket or a web server. CloudFront uses a distribution that gives you a special domain name to access this cached content. Cached data stays at edge locations for a defined time called TTL. If content changes before TTL expires, you can force an update using cache invalidation. CloudFront supports HTTPS by default for secure delivery. It also sends performance metrics to CloudWatch for monitoring. Overall, CloudFront improves speed, performance, and user experience globally.

What CloudFront Actually Stores

CloudFront caches objects, identified by:
URL / file name (e.g. /index.html)
Query strings / headers (if configured)

CloudFront does NOT check file content or timestamps at origin by default.

#### Catche Invalidation (cost applied)

Case 1️⃣ — Without Invalidation (Only TTL)
You update /index.html in S3
CloudFront still has /index.html cached
User requests /index.html
CloudFront says:
“TTL not expired → serve cached copy”
Old content is served, even though origin changed
📌 CloudFront never asks the origin until TTL expires.

Case 2️⃣ — With Cache Invalidation
You update /index.html in S3
You send Invalidate /index.html request to CloudFront
CloudFront deletes the cached object from all edge locations
Next user request comes
CloudFront says:
“Cache miss — I don’t have this file”
CloudFront fetches the file again from origin
New version is cached and served
👉 Old cache is gone only because YOU deleted it

#### Object Versioning (Free)
Change the Object Version (File Versioning)
Changing object version means changing the file name itself whenever content changes.
How it works:
Old file: app.js
New file: app.v2.js
CloudFront sees this as a brand-new file
No old cache exists → fresh content served immediately
Example:
style.css → style.2026.css
image.png → image_v2.png
📌 Good for: regular updates, large apps, static assets
📌 Best practice in real production

Invalidation deletes old cache for the same URL, versioning changes the URL so cache automatically misses and updates.

### Lambda@Egde
Lambda@Edge is an AWS service that lets you run serverless code at CloudFront edge locations to customize and process requests and responses closer to users, reducing latency(faster response) and improving performance.

#### CloudFront receives a request from a user

The request goes to the nearest edge location.
CloudFront first checks its cache for the requested URL.
Lambda@Edge can trigger BEFORE cache is used (depending on event)
CloudFront supports 4 types of Lambda@Edge triggers:

#### Event	When it runs	Purpose-
Viewer Request	runs When request arrives at edge purpose is to	Modify request, authentication, URL rewrite
Origin Request	runs When content is not cached and CloudFront goes to origin	purpose is to Add headers, customize origin request
Origin Response	runs When CloudFront gets content from origin	purpose is to Modify response before caching or sending to viewer
Viewer Response	runs when Before sending cached or origin content to viewer	purpose is to Add headers, tracking, personalization

#### Integration with caching (TTL)
CloudFront serves cached content if TTL not expired.
Lambda@Edge can run even for cached content depending on trigger:
Viewer Request / Viewer Response → runs every time user requests content at edge, even if cached.
Origin Request / Origin Response → runs only when content is fetched from origin.

#### Why this is powerful
You can modify requests and responses at the edge without touching your origin server.
You can customize cached content for users globally.
Reduces latency → faster user experience.

### Global Accelerator
AWS Global Accelerator is a service that makes your application faster and more reliable for users around the world. It routes user traffic to the nearest AWS edge location instead of sending it through the slow public internet. From there, the traffic travels on the AWS private backbone network, which is faster and more stable. This reduces latency, packet loss, and connection issues. Global Accelerator is mainly used for global applications that need consistent performance for all users.

##### AWS private backbone network
AWS private backbone is AWS’s own global network made of private fiber-optic cables that connect AWS regions, edge locations, and data centers worldwide.
This network is not the public internet and is fully owned, controlled, and optimized by AWS.
What “private” actually means here
###### It does NOT mean:
Private IPs only
VPN only
Hidden from everyone
###### It DOES mean:
AWS owns the cables
AWS controls routing
No ISP guessing or congestion
Predictable speed and latency

How it works (simple flow)
User request reaches nearest AWS edge location(still public internet)
Traffic enters the AWS private backbone
Request travels securely and quickly to the AWS region
Response comes back the same optimized way

#### User → Nearby Edge Location → AWS Private Backbone → Server (Region)

Global Accelerator is not useed everywhere, even though it’s fast because-
- It costs extra Global Accelerator is a paid service Charges for: Accelerator hours Data transfer and Not worth it for small or regional apps
- Not needed for cached/static content as CloudFront already solves this CloudFront: Caches content at edge and Reduces origin traffic Global Accelerator does NOT cache Using GA for static files = waste of money
- Regional users don’t need it If users are mostly in one region Normal ALB + Route 53 is enough Latency gain is minimal

Where Global Accelerator is used
- Global applications with users worldwide
- Latency-sensitive apps Gaming, Live streaming, Trading platforms, Real-time APIs
- Multi-region active-active architectures Applications needing fast failover
- When availability and reliability matter more than cost

```
How to choose (easy decision rule)
Scenario	                    Use
Static content	              CloudFront
Dynamic global traffic	      Global Accelerator
Regional app	                ALB / Route 53
Need caching	                CloudFront
Need fast routing	            Global Accelerator
```
