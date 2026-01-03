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
 


