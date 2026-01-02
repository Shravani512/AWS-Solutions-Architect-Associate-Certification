- VPC
- Elastic IP
- Firewalls
  1. Stateless firewall
  2. Statefull firewall
 
***** Stateless Firewall
A stateless firewall filters network traffic only based on rules like source/destination IP address, port number, and protocol.
It does not remember previous packets or track ongoing connections.
It is used where simple and fast filtering is needed and traffic inspection is not complex.

Real-time example:
AWS Network ACL (NACL)

Example rule:
“Allow inbound traffic on port 80 from any IP”
Each packet is checked independently, without knowing if it is part of an existing connection.

***** Statefull Firewall
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

*** Important
NACLs are stateless firewalls that operate at the subnet level and control inbound and outbound traffic entering or leaving the subnet. They do not filter traffic between instances within the same subnet. Security Groups are stateful firewalls attached to instances and they control traffic between instances, even within the same subnet.

NACL → Rules are evaluated in order (rule number) 
Security Group → All rules are evaluated together

***** NACL: “Rules are evaluated in order (rule number)”
What this means 👇

Every NACL rule has a rule number (100, 110, 120, etc.)
AWS checks rules from the smallest number to the largest
First matching rule wins
After a match, no more rules are checked

Example 🚦 (NACL)
Rule No	Type	Source	Action
100	HTTP	0.0.0.0/0	ALLOW
110	HTTP	1.2.3.4/32	DENY

Incoming request from 1.2.3.4 on port 80:

➡ AWS checks rule 100 first
➡ It matches → ALLOW
➡ Rule 110 is never checked

🔴 Even though there is a DENY rule, traffic is allowed because lower rule number matched first.

***** Security Group: “All rules are evaluated together”
What this means 👇

Security Groups do NOT have rule numbers

There is no order
AWS checks all rules at once
If any rule allows the traffic → traffic is allowed
If no rule allows it → traffic is denied (implicit deny)

Example 🔐 (Security Group)
Type	Port	Source
HTTP	80	0.0.0.0/0
HTTP	80	1.2.3.4/32

Incoming request from 1.2.3.4 on port 80:

➡ AWS checks all rules
➡ Finds a matching ALLOW
➡ Traffic is allowed

There is no concept of deny or priority here.
