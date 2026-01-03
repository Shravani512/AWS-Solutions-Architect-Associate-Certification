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

#### Direct Connect

Architecture
![]()
AWS Direct Connect Architecture allows you to establish a dedicated, private network connection between your on-premises data center and AWS. Instead of using the public Internet, traffic flows through a high-speed, low-latency link, providing more consistent and secure connectivity. In a typical setup, your on-premises router connects to a Direct Connect location via a physical connection, which then links to one or more VPCs in AWS through Virtual Interfaces (VIFs). There are usually two types of VIFs: private VIF for accessing VPC resources and public VIF for accessing AWS public services like S3. Direct Connect can be combined with VPN as a backup to ensure high availability. Overall, this architecture improves performance, security, and reliability for hybrid cloud setups.

#### VPC Peering
Earlier, if you had two different VPCs, you could not connect their subnets directly. Each VPC was isolated, so resources in one VPC (like EC2 instances or databases) couldn’t talk to resources in another VPC.

That’s where VPC Peering came into the picture. It creates a private connection between two VPCs, allowing their subnets to communicate with each other using private IPs, without going over the Internet. Now, instances in one VPC can access instances or services in another VPC securely and directly.
