## CloudFormation

- CloudFormation → AWS infrastructure creator
- Terraform → Multi-cloud infrastructure creator
- Ansible → Server configuration tool

CloudFormation and Terraform does same work only difference is Terraform in suitable for multicloud like (AWS, Azure, GCP etc and CloudFormation creates infra only for AWS)
Ansible is for configuration of servers like terraform/CloudFormation creates EC2 and Ansible install Nginx in it.

**Companies often use:**

Terraform + Ansible
or
CloudFormation + Ansible

**CloudFormation Explanation**

AWS CloudFormation is a service that lets you create and manage AWS infrastructure using code files (templates) instead of manual setup in the console.

You define resources like:
EC2, S3, VPC, RDS
inside a template (JSON/YAML), and AWS automatically builds them for you.

**Why CloudFormation Is Used**

Before IaC tools existed, infrastructure was created:
manually in console
with scripts
or via runbooks

Problems with Old Method

❌ Human errors
❌ Different environments not identical
❌ Hard to recreate setup
❌ No version history
❌ Time-consuming

CloudFormation Solves This By

Automating setup
Using code templates
Making infrastructure reproducible
Tracking changes
Managing everything as one unit

**How CloudFormation Works**

1. You write a template (JSON/YAML)
2. Upload template to CloudFormation
3. CloudFormation reads template
4. It creates resources automatically
Resources are grouped as a Stack

Stack = collection of resources managed together

```
Web App Stack
 ├── EC2
 ├── Load Balancer
 ├── Security Group
 └── Database
```

**Updating Infrastructure (Very Powerful Feature)**

Instead of manually editing resources:

- Edit template
- Upload updated template
- CloudFormation shows Change Set (preview of changes)
- You approve
- AWS updates safely

👉 No manual edits needed
👉 No accidental deletions

**Key Features**

1. Infrastructure as Code (IaC)
Infrastructure defined as code → stored in Git → version controlled.
2. Consistent Deployments
Same template = identical environments.
Example: Dev Test Prod All match exactly.
3. Version Control Friendly
You can track: who changed what, when, why
4. Resource Tracking via Stacks
You manage multiple resources as one unit instead of individually.
Delete stack → everything deletes together.
5. Efficiency + Cost Saving
Automation reduces: manual work, mistakes and deployment time

**Real-World Example**

A company launches 10 identical app environments for clients.
**- Without CloudFormation:**
setup takes hours each
mistakes possible

**- With CloudFormation:**
run template 10 times
done in minutes

## CDK

What AWS CDK Is

AWS CDK (Cloud Development Kit) is an Infrastructure-as-Code tool that lets you create AWS resources using programming languages like:

Python, JavaScript / TypeScript, Java, C# (.NET)

Instead of writing JSON/YAML templates, you write real code.

👉 Then CDK automatically converts your code into CloudFormation templates and deploys them.

**Why CDK Is Used**

Problem with Traditional IaC

Tools like CloudFormation templates:

hard to write, hard to debug, no loops/functions, repetitive code, poor reusability

Example problem:
Creating 10 similar resources = copy-paste 10 blocks.

**- CDK Solution**

CDK lets you use programming features:
loops, variables, functions, classes, conditions

So instead of repeating 10 times:

```
for i in range(10):
    create_bucket()
```

**How CDK Works (Step-by-Step)**

1. You write infrastructure in code (Python/TS/etc.)

2. You run:
```
cdk synth
```
3. CDK converts your code → CloudFormation template
4. Run:
```
cdk deploy
```
5. CloudFormation creates resources

👉 Important:
CDK does NOT directly create resources
CloudFormation still does deployment.

**Features**

1. Transparency & Predictability

Infrastructure defined as code = predictable deployments.
Same code → same infra every time.

2. Code Reusability

You can write reusable infrastructure modules.
Example:
MyCompanySecureBucket()

Use it in all projects.

3. Rich AWS Construct Library

AWS provides ready-made constructs for almost every service:
EC2, VPC, RDS, S3, Lambda

So you don’t build everything from scratch.

4. Automated Synthesis
Command:
```
cdk synth
```
automatically converts code → CloudFormation template.
No manual template writing.

5. Environment Agnostic Deployments

You can deploy same code to:
dev, staging, prod
Just change parameters.

**CDK + CI/CD Integration (Real DevOps Usage)**

Typical pipeline:
```
Developer updates CDK code
        ↓
Push to Git repo
        ↓
CI pipeline runs
        ↓
cdk synth
        ↓
Tests run
        ↓
cdk deploy
        ↓
Infrastructure updated
```

CDK combines power of programming languages with infrastructure automation.

You can:
use libraries
write functions
create reusable classes
run tests

**Real-World Example**

Company wants a standard VPC architecture for all apps.
Instead of writing template every time:
They create:
```
StandardVPC()
```
Now every project just calls it once.
Done.
```
CDK = Code → Synth → Template → Deploy → Stack
```

## CloudWatch

What CloudWatch Is
AWS CloudWatch is a monitoring and observability service that collects data from AWS resources and applications and helps you:

monitor performance
detect problems
trigger alerts
automate actions

👉 Think of it as the health monitoring system of your cloud infrastructure.

**Lifecycle / Flow of CloudWatch (End-to-End)**

Step-by-step lifecycle:

1. AWS services or apps generate data
CPU usage, logs and errors

2. CloudWatch collects data automatically or via SDK

3. CloudWatch processes and stores data

4. You define alarms or dashboards

5. If threshold breached:
alarm triggers
notification sent
automation runs (optional)

6. You analyze logs or metrics to fix issues
```
Resource → Metrics/Logs → CloudWatch → Alarm → Notification/Action
```
**Compared to Tools Like Prometheus / Datadog**

CloudWatch is:
native, integrated and simpler setup

External tools are:
cross-cloud, more customizable and more complex

**Key Components**

1️⃣ Metrics

Metrics are numeric performance data points.
Examples:
CPU %, memory usage,latency, request count

👉 Metrics answer: “Is my system healthy?”

2️⃣ Alarms

Alarms watch metrics and trigger when thresholds are crossed.
Example:
If CPU > 80% for 5 mins → send alert

Actions can include:
SNS notification
Auto Scaling
Lambda trigger

3️⃣ Logs

Logs are detailed records of events.
Examples:
Error logs, request logs and system logs
Logs help answer:

“Why did it fail?”

4️⃣ Events (EventBridge)

EventBridge detects state changes and triggers automation.
Example:
New file uploaded → trigger Lambda
Instance stopped → notify admin
👉 It’s event-driven automation.

5️⃣ Dashboards

Custom dashboards show all monitoring data in one place.
Example dashboard shows:
CPU, memory, error rate and requests/sec

6️⃣ Logs Insights
Advanced query tool for logs.
You can search logs like SQL:
find all errors in last 10 mins

**Real-World Example**

Suppose your website becomes slow.

CloudWatch shows:
CPU spike
memory usage high
request count surge

Alarm triggers → Auto Scaling launches new servers → website stabilizes.

Without CloudWatch → users complain before you even know.
```
CloudWatch metrics + logs + alarms + dashboards.
```

## X-Ray

AWS X-Ray is a distributed tracing service that tracks requests as they travel through your application and its services.

👉 It shows:
- where time is spent
- where failures occur
- which service is slow

Think of it as:
A GPS tracker for every request in your system.

**Why X-Ray Is Used (Problem It Solves)**
**- Problem in Modern Systems**
Modern apps use many services:
APIs
databases
microservices
queues
storage

**- If something is slow:**
❌ you don’t know which service caused it
❌ logs don’t show full flow
❌ debugging takes hours

**X-Ray Solution
X-Ray tracks the entire journey of a request across services.**
So you can instantly see:
which component is slow
where error occurred
exact latency of each step

**How X-Ray Works (Lifecycle Flow)**

Step-by-step lifecycle:

1. User sends request to app
2. X-Ray SDK records request
3. Request travels across services
4. Each service adds its segment data
5. X-Ray collects all segments
6. X-Ray builds full trace
7. Console shows visual trace map

One-Line Flow
```
Request → Segments → Trace → Visualization → Diagnosis
```
**Key Concepts**

1️⃣ Trace

A trace = full journey of one request.
Includes: all services, timing and status
👉 One request = one trace

2️⃣ Segment

A segment = single step inside a trace.
Examples: Lambda execution, DB call, API processing
👉 Many segments make one trace.

3️⃣ Subsegment (Advanced Concept)

Smaller operations inside a segment.
Example:
Segment = Lambda execution
Subsegments: DB query, API call, file read

**Real-World Scenario**

Problem:
Users complain login is slow.

**- CloudWatch shows:**
CPU normal
memory normal
**Still slow.**

**- X-Ray shows:**
Database query taking 4 seconds.
Now you know root cause instantly.

## AWS Health Dashboard

AWS Health Dashboard is a service that shows the real-time health status of AWS services and alerts you if any issue or maintenance affects your resources.

**Key Idea**

It tells you whether a problem is caused by your application or AWS itself.

**Two Types of Events**

Public events → Affect many customers (e.g., region outage)
Private events → Affect only your account resources

Integration

**It can send alerts via:**
SNS
CloudWatch
EventBridge
So your team is notified automatically.

**Important**

- My server: the cloud resources you created and control in AWS (like your EC2, database, app).
- AWS server: the underlying AWS infrastructure that Amazon manages (data centers, hardware, core services).
```
If your server crashes → CloudWatch tells you
If AWS server crashes → Health Dashboard tells you
```

## Prometheus

**What it is:**
Prometheus is a tool that collects numbers about your system.
Example:
It collects CPU %, memory usage, request count, errors.

In one line:
Prometheus = data collector

## Grafana

**What it is:**
Grafana is a tool that shows those numbers as graphs.
Example:
It turns Prometheus data into charts, dashboards, alerts.

In one line:
Grafana = data viewer

**Prometheus stores metrics → Grafana shows metrics**

**Cloudwatch vd Prometheus**

CloudWatch is designed mainly for monitoring AWS resources.
Prometheus can monitor anything — AWS, other clouds, on-prem, containers, etc.

- AWS-native monitoring (CloudWatch) → It is built by AWS mainly to monitor AWS services like EC2, Lambda, RDS. It fits best when your app runs inside AWS.

- Platform-independent monitoring (Prometheus) → It is not tied to any company or cloud. It can monitor anything — AWS, Azure, GCP, local servers, Kubernetes, etc.


**Why big tech companies still prefer Prometheus**

Imagine a fast-growing tech company that first built its app entirely on AWS, so they happily used CloudWatch because it was simple, automatic, and required zero setup. Everything worked fine at the start. But as the company scaled, they added Kubernetes, on-prem servers, and another cloud provider for cost and performance reasons. Suddenly, CloudWatch could easily monitor only the AWS part, while the rest needed separate tools. Managing multiple monitoring systems became messy and slow during outages. So the engineers switched to Prometheus, which could monitor all systems in one place and let them create deep custom metrics and powerful queries. It required more setup, but once running, it gave them full control, unified visibility, and flexibility — which is why large tech companies often choose Prometheus when their infrastructure becomes complex and multi-cloud.

## AWS Resource Access Manager (RAM)

AWS RAM is a service that allows you to securely share AWS resources across multiple AWS accounts or within an organization without duplicating them.

**Why is it used? (Problem it solves)**

In multi-account setups (common in companies):

**- Without RAM**
Each account creates its own resources
Duplicate infrastructure → higher cost
Complex networking setups (VPC peering/VPN)
Harder management
Problem: Resource fragmentation across accounts.

**How it works (Lifecycle Flow)**

Step-by-step process

Create resource in Account A
Create Resource Share
Select resource
Choose accounts/OUs to share
Recipient accepts invite
Resource becomes usable

👉 Flow logic:
Create → Share → Accept → Use

**Common Use Cases**

1. Subnet sharing
Networking team owns VPC
App teams launch EC2 inside shared subnet

2. Transit Gateway sharing
Central networking account manages routing

3. License sharing
Share purchased licenses across accounts

4. S3 sharing
Multiple accounts access same storage

**Real-World Example**

Company structure:
```
Networking Account → owns VPC + Subnets
Application Account → deploys EC2
Security Account → monitors traffic
```
Using RAM:

Networking team shares subnet
App team launches EC2 in it
Security team monitors

👉 All using same infrastructure.

**RAM shares resources, not permissions or accounts.**

**When should you use RAM?**

Use it when:

You have multiple AWS accounts
Need shared infrastructure
Want centralized networking
Want cost optimization

AWS RAM enables secure cross-account resource sharing so organizations can centrally manage infrastructure and avoid duplicating resources across multiple AWS accounts.

**RAM = “One resource, many accounts”**