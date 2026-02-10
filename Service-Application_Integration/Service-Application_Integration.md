## Autoscaling

Auto Scaling is an AWS service that automatically increases or decreases EC2 instances based on application demand, ensuring high availability and cost efficiency without manual intervention.

**Autoscaling Group**

An Auto Scaling Group is a logical group of EC2 instances that maintains a specified number of instances and automatically replaces unhealthy instances.

An Auto Scaling Group is like a manager that controls:
- how many EC2 instances should run
- when to add more
- when to remove extra ones

Every ASG has 3 limits:
Minimum- Lowest number of instances (never go below this)
Desired- Ideal number of instances right now
Maximum- Highest number of instances allowed

App starts with 2 instances
Traffic increases → ASG can go up to 10
Traffic reduces → ASG never goes below 2

**Auto-Healing**
Auto-healing is the ability of an Auto Scaling Group to detect unhealthy EC2 instances, terminate them, and launch replacements automatically.
Problem:
One EC2 instance crashes 

What ASG does:
Detects unhealthy instance
Terminates it
Launches a new one automatically

**Types of Scaling Policies**
A scaling policy defines how and when an Auto Scaling Group adds or removes EC2 instances.

AWS supports 3 ways to scale:
1. Manual Scaling (Basic): 
2. Dynamic Scaling:
        - Dynamic Scaling has 3 types
        1. Target Tracking Scaling
        2. Simple Scaling
        3. Step Scaling
3. Scheduled Scaling

- Manual Scaling:

Manual scaling allows users to manually change the desired capacity of an Auto Scaling Group.

- Dynamic Scaling

Dynamic scaling automatically adjusts EC2 instances based on real-time CloudWatch metrics such as CPU utilization or network traffic.

Dynamic scaling types:
1.  Target Tracking Scaling
Target tracking scaling maintains a specific metric value (for example, average CPU utilization at 50%) by automatically scaling in or out.
“Maintain this value” approach
Example:
Target CPU = 50%
CPU > 50% → add instances
CPU < 50% → remove instances

You run a shopping app
Traffic increases → CPU becomes 70%
ASG adds new EC2
CPU comes back near 50%

2. Simple Scaling
Simple scaling uses CloudWatch alarms to trigger a fixed scaling action, such as adding or removing a specific number of instances.

Alarm-based scaling
Example:
If CPU > 70% → add 2 instances
If CPU < 30% → remove 1 instance
Triggered by CloudWatch alarms

3. Step Scaling
Step scaling adjusts the number of EC2 instances based on different metric ranges, allowing more precise scaling actions.
Scale based on how much load increases
Example:
CPU 60–70% → add 2 instances
CPU 70–85% → add 3 instances
CPU 85–100% → add 5 instances

**Launch Template(Blueprint for EC2)**
Whenever ASG launches a new EC2, it needs instructions.
That instruction set is called a Launch Template.

It contains:

AMI (OS), Instance type (t2.micro, t3.small, etc.), Security groups, Subnets, IAM role, User data scripts (startup commands)

Why Launch Template is important:
Same configuration every time
Easy version updates
No manual setup

**Integration with Other AWS Services**
Auto Scaling integration refers to how Auto Scaling works with ELB, CloudWatch, and SNS to manage traffic, monitor health, and notify users.
Auto Scaling doesn’t work alone it works with-
1. Elastic Load Balancer (ELB): An Elastic Load Balancer distributes incoming traffic across multiple EC2 instances to improve availability and fault tolerance.
2. CloudWatch:Amazon CloudWatch is a monitoring service that collects metrics and triggers alarms used by Auto Scaling to make scaling decisions.
3. SNS: Amazon Simple Notification Service (SNS) sends notifications about Auto Scaling events, such as instance launches or terminations.

**Full End-to-End Scenario (Real World)**
Online Shopping App 🛒

- Normal time → 2 EC2 instances
- Sale starts → traffic spikes
- CPU goes above 50%
- Auto Scaling:
            - Adds new EC2s
            - Load balancer distributes traffic
- One EC2 crashes 
- ASG replaces it automatically
- Sale ends → traffic drops
- ASG removes extra instances
- You pay only for what you used 

## Elastic Load Balancing

Elastic Load Balancer (ELB) is an AWS service that automatically distributes incoming traffic across multiple servers (EC2, containers, Lambda).
It prevents any single server from getting overloaded, improving performance and availability.
ELB also checks server health and sends traffic only to healthy instances.

**Scaling approaches**

1. Vertical Pod Autoscaling(VPA)
Vertical scaling means increasing the power of a single server by adding more CPU, RAM, or storage.

2. Horizontal scaling
Horizontal scaling means adding more servers and sharing traffic among them.

The real challenge without a load balancer

Even if you launch multiple EC2 instances, your domain name
mywebsite.com can still point to only one IP address.

So theoretically:

Elastic Load Balancing (ELB) is an AWS-managed service that:
Acts as a single entry point for users
Automatically distributes incoming traffic across multiple backend resources
Works across multiple availability zones
Improves availability, scalability, and fault tolerance

**How ELB works internally**

1. Listener
A listener is a rule that tells the load balancer:
Which protocol to listen on (HTTP, HTTPS, TCP, etc.)
Which port to listen on (80, 443, 8080)

2. Target Group
A target group is a logical group of backend endpoints where traffic is sent.
Targets can be:
EC2 instances, IP addresses, Lambda functions, Even other load balancers

**Health checks**
Health checks allow the load balancer to:
Periodically test backend targets
Route traffic only to healthy targets
Automatically stop sending traffic to failed ones

**Public vs Private Load Balancers**

1. A public load balancer:

Is internet-facing
Has a public DNS name
Accepts traffic from users worldwide

2. A private load balancer:

Is internal to the VPC
Has no public access
Used for backend services

**Cross-Zone Load Balancing**

With cross-zone load balancing enabled:

Traffic is evenly distributed across all instances in all AZs
Without it:
Each AZ handles only its own traffic

**Types of AWS Load Balancers**

1. Application Load Balancer (ALB)
ALB operates at Layer 7 (Application Layer) and understands:
HTTP
HTTPS
URLs
Headers
Methods
It supports content-based routing.

Examples:

I. Host-based routing
blog.mywebsite.com → Blog service
api.mywebsite.com → API service

II. Path-based routing
/login → Auth service
/orders → Order service

III. Method-based routing
GET → Read servers
POST → Write servers

IV. Header / Query routing
x-environment=staging → Staging backend
?category=books → Book service

2. Network Load Balancer (NLB)

NLB operates at Layer 4 (Transport Layer):

Works with TCP, UDP, TLS
Extremely fast
Handles millions of requests
Uses static IP addresses

NLB is a high-speed tunnel:

Doesn’t inspect the packet
Just forwards it as fast as possible
Perfect for performance-critical systems

**Elastic Load Balancing allows you to:**

Accept traffic at one stable entry point
Distribute it intelligently
Detect failures automatically
Scale smoothly
Remain highly available even during failures

## API Gateway

API Gateway is used because modern applications are not built using just one server or one service anymore. Behind a single app, many different services are working together, such as Lambda for logic, EC2 or ECS for applications, DynamoDB for data, S3 for files, Cognito for login, and Step Functions for workflows. If we allow users or frontend apps to talk directly to all these services, the system becomes messy and unsafe: users must remember many different URLs, security must be handled separately for each service, traffic can overload systems, there is no single place to monitor requests or errors, and even small backend changes can break the application. API Gateway fixes this by acting as one single, smart entry point for everything. Clients send all requests to API Gateway only. API Gateway first checks who is making the request and whether they are allowed, then controls how much traffic is allowed so backends do not crash, checks and adjusts request and response formats if needed, and forwards the request to the correct backend service. If required, it can even combine data from multiple services into one response. At the same time, API Gateway tracks request counts, errors, and delays using CloudWatch, helps manage different API versions and environments like development, testing, and production, automatically creates API documentation, securely connects to private services inside a VPC, improves performance using caching and rate limits, and works smoothly with all major AWS services. In simple terms, API Gateway hides the complexity of backend systems, keeps everything secure and controlled, scales reliably as traffic grows, and makes APIs easy for developers to use. In story form, it is like a smart and secure front desk that handles all visitors while internal teams are free to change and improve their systems without users ever noticing.

**API Types supported by API Gateway**

1. REST API: Traditional request-response APIs with rich features like caching and transformations; best for full-control, synchronous CRUD systems.

2. HTTP API: Lightweight, faster, and cheaper APIs mainly for simple routing to Lambda or HTTP backends.

3. WebSocket API: Two-way, real-time communication where server and client can push messages independently (chat, live updates).

**Authentication / Authorization Types**

1. IAM: For AWS services and internal AWS-to-AWS access.

2. Cognito: For application users using JWT tokens.

3. OAuth / Custom Authorizers: For custom or third-party identity systems.

**Backend Integration Types**

1. Lambda: Serverless business logic.

2. EC2 / ECS / Beanstalk: Application servers.

3. HTTP endpoints: External or internal services.

4. AWS services: DynamoDB, S3, Step Functions, Kinesis, RDS (via backend).

**Deployment / Lifecycle Types**

1. Stages: dev, test, prod (same API, different environments).

2. Versions: v1, v2 running in parallel without breaking clients.

**Traffic Control Types**

1. Rate limiting: Requests per second control.

2. Throttling: Slow down or block excess traffic.

3. Quotas: Usage limits per API key or user.

**Caching (REST APIs only)**

1. TTL-based caching: Serve repeated requests without hitting backend.

**API Visibility Types**

1. Public APIs: Accessible over the internet.

2. Private APIs: Securely routed to VPC-internal resources only.

## AppFlow

What problem AppFlow solves 

In real companies, data is scattered everywhere.
Sales team uses Salesforce
Support team uses Zendesk
HR uses some HR tool
Engineers store data in AWS (S3, Redshift, DynamoDB)

All this data lives in different apps and doesn’t talk to each other.

So people end up:
Downloading CSV files
Copy-pasting data
Writing scripts
Sending files on email
This is slow, error-prone, and not secure.

**What Amazon AppFlow is-**

Amazon AppFlow is a service that automatically moves data from one application to another — safely and without writing code.

You tell AppFlow:
From where to take data
To where to send data
When to send it

And AWS does the rest.

**What AppFlow connects** (big picture)**

AppFlow mainly connects:

SaaS applications (Salesforce, Slack, Zendesk, ServiceNow, etc.)
AWS services (S3, Redshift, RDS, DynamoDB)

It acts as a bridge between them.

**Core Building Blocks of Amazon AppFlow**

1. Connector
The secure “login bridge” that lets AppFlow talk to a source or destination app (like Salesforce, Slack, or S3).

2. Flow
The rule that says what data to move, from where to where, and how it should look.

3. Trigger
The timer or event that decides when the data transfer should run (manual, event-based, or scheduled).

That’s it.
Connector = connection, Flow = data movement logic, Trigger = timing.

**Examples**

1. Example 1: Salesforce → Redshift
Whenever new sales data is created:
AppFlow moves it to Redshift
Analysts run reports

2. Example2: Slack → S3
Every day:
AppFlow copies Slack messages
Stores them in S3 for analysis

3. Example 3: Zendesk → S3
Support tickets:
Filtered by case number
Stored for trend analysis

4. Example 4: Multiple apps → S3
Salesforce + Zendesk + ServiceNow:
All data collected
Stored in S3 as a data lake

## Amazon SNS (Simple Notification Service)

**What it is:**
SNS is a push-based messaging service used to send the same message to many subscribers at once. When something happens, SNS immediately notifies everyone who is interested. Subscribers can be emails, SMS, Lambda, SQS, or HTTP endpoints. SNS does not store messages for long — it just broadcasts and moves on. It’s best for alerts, notifications, and fan-out patterns.

Scenario:
An order fails → SNS sends email to admin, SMS to on-call engineer, and message to SQS simultaneously.

Memory hook:
👉 SNS = “Shout once, everyone hears.”

## Amazon SQS (Simple Queue Service)

**What it is (easy):**
SQS is a queue-based service that stores messages until they are processed. Producers send messages, consumers pick them up one by one. If a consumer is slow or fails, messages wait safely in the queue. This decouples systems and smooths traffic spikes. Nothing is pushed — consumers pull messages when ready.

Scenario:
Users place orders → orders go into SQS → workers process them slowly without crashing.

Memory hook:
👉 SQS = “Wait in line, get processed safely.”

## Amazon MQ

**What it is (easy):**
Amazon MQ is a managed traditional message broker (like ActiveMQ or RabbitMQ). It is mainly for legacy or enterprise systems that already use protocols like JMS, AMQP, or MQTT. Unlike SQS/SNS, you manage queues, topics, and brokers more closely. AWS manages servers, but the messaging style is old-school.

Scenario:
A banking system already using ActiveMQ moves to AWS without rewriting applications.

Memory hook:
👉 Amazon MQ = “Old messaging, but hosted by AWS.”

## Amazon EventBridge

**What it is (easy):**
EventBridge is an event routing service used to connect services when something happens. Producers emit events, EventBridge routes them to targets using rules. Services don’t know about each other — they just react to events. It’s ideal for event-driven architectures and SaaS integrations.

Scenario:
User signs up → event sent → one service sends email, another creates user record, another triggers analytics.

Memory hook:
👉 EventBridge = “When this happens, do that.”

**Important**

- SNS → Notify many immediately
- SQS → Queue work safely
- Amazon MQ → Legacy enterprise messaging
- EventBridge → Event-driven automation

SQS = Simple, serverless, AWS-native queue
Amazon MQ = Managed traditional message broker for legacy apps

SQS and MQ solve the same problem, but SQS is built for cloud-native apps, while MQ is built for legacy enterprise apps.

**Cloud-native apps**
Applications designed from day one for the cloud, using managed services, auto-scaling, APIs, and serverless (example: microservices using Lambda, SQS, DynamoDB).

**Legacy enterprise apps**
Old or traditional applications originally built for on-premise servers, fixed infrastructure, and classic protocols (example: monolith apps using JMS, ActiveMQ, MQ).

## Amazon Simple Email Service (SES)

**What it is:**
Amazon SES is a cloud service used to send emails automatically from applications.

Real-world scenario:
When you sign up on a website and instantly get a verification email, password reset link, or order confirmation, SES is sending that email reliably at scale.

Why it matters:
It lets apps send millions of emails cheaply, securely, and without managing mail servers.

## AWS Step Functions

**What it is:**
Step Functions is a service to coordinate multiple AWS services in a step-by-step workflow.

Real-world scenario:
Placing an order → charge payment → update inventory → send email → handle failure if payment fails — Step Functions controls this flow automatically.

**- SES = automatic emails from apps**
**- Step Functions = manager that controls step-by-step workflows**

## Amazon Simple Workflow Service (SWF)

**What it is:**
Amazon SWF is a service used to run long-running business workflows where humans or systems may take time to respond.

Real-world scenario:
A loan approval process → application review → manager approval → background check → final approval, where each step may take hours or days.

Why it matters:
It reliably tracks workflow state even if tasks take a long time or systems temporarily fail.

## Amazon Managed Apache Airflow (MWAA)

**What it is:**
MWAA is a managed service to schedule and orchestrate data pipelines using Apache Airflow.

Real-world scenario:
Every night: extract sales data → clean it → load into Redshift → run reports → send results to teams.

Why it matters:
It automates complex data workflows without managing Airflow servers, using code-based DAGs.

**- SWF = long human-involved workflows**
**- Airflow (MWAA) = scheduled data pipelines**

## Design: Scalable AWS Application using Application Integration Services
Problem statement 

Design a highly scalable web application (like an e-commerce or booking app) that can handle traffic spikes, background processing, notifications, and failures.

**Answer**
**High-Level Architecture Flow**

A user sends a request from a web or mobile app which first reaches API Gateway (secure front door). API Gateway forwards the request to an Application Load Balancer (ELB), which distributes traffic across multiple EC2 instances. Auto Scaling increases or decreases EC2 servers based on load to handle traffic spikes.

For time-consuming tasks (order processing, payment, image processing), the app pushes messages to SQS, so the system stays responsive. SNS sends notifications to multiple services at once (email, SMS, Lambda). EventBridge captures business events like order placed and routes them to the right consumers.

Step Functions orchestrates complex workflows like payment → inventory → shipping → notification. SES sends emails such as OTPs and invoices. AppFlow syncs data with SaaS apps (CRM, Salesforce). Managed Apache Airflow (MWAA) runs scheduled jobs like daily reports. Amazon MQ is used only if legacy systems need JMS/AMQP messaging.

**timeline-style explanation** 

T0 – User clicks action in web/mobile app

T1 – Request enters API Gateway (auth, throttling)

T2 – ALB forwards request to a healthy EC2

T3 – Auto Scaling adds/removes EC2 based on load

T4 – Application code runs on EC2

T5 – Heavy work sent to SQS / Amazon MQ

T6 – Multi-step logic handled by Step Functions / SWF

T7 – Business events routed via EventBridge

T8 – Notifications sent using SNS → SES

T9 – Data sync & batch jobs via AppFlow / MWAA (Airflow)