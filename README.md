# 9. AWS Auto Scaling 

## Introduction

**AWS Auto Scaling** automatically increases or decreases the number of EC2 instances running your application based on demand.

Instead of manually creating EC2 instances when traffic increases, an **Auto Scaling Group (ASG)** can automatically launch new instances.

When traffic decreases, it can terminate unnecessary instances.

### Basic Architecture

```text
Users
  ↓
ALB
  ↓
EC2 ←→ EC2 ←→ EC2
  ↑
Auto Scaling Group
```

With Auto Scaling, the number of EC2 instances can automatically change according to application demand.

---

# 1. What is Auto Scaling?

Auto Scaling is the process of automatically adjusting computing resources according to workload or demand.

For EC2, an Auto Scaling Group can:

* Launch new EC2 instances
* Terminate unnecessary EC2 instances
* Replace unhealthy EC2 instances
* Maintain a desired number of instances
* Scale out when demand increases
* Scale in when demand decreases
* Distribute instances across Availability Zones

### Example

Suppose your application initially has:

```text
EC2-1
EC2-2
EC2-3
```

If traffic increases:

```text
EC2-1
EC2-2
EC2-3
EC2-4
EC2-5
```

Auto Scaling can launch additional instances.

If traffic decreases:

```text
EC2-1
EC2-2
EC2-3
```

Auto Scaling can remove unnecessary instances.

---

# 2. Why Do We Need Auto Scaling?

Imagine an application running on only one EC2 instance.

```text
Users
  |
  v
EC2 Server
```

If thousands of users access the application at the same time:

* CPU utilization can increase.
* Memory usage can increase.
* Application performance can decrease.
* Requests can become slow.
* The EC2 instance can become overloaded.
* If the server fails, the application may become unavailable.

Instead, we can use multiple EC2 instances:

```text
                    Users
                      |
                      v
              +---------------+
              | Load Balancer |
              +---------------+
                /      |      \
               /       |       \
              v        v        v
            EC2-1    EC2-2    EC2-3
```

If traffic increases, Auto Scaling can add more instances.

If traffic decreases, Auto Scaling can remove unnecessary instances.

---

# 3. Main Benefits of Auto Scaling

## 3.1 Automatic Scaling

Instances can automatically increase or decrease based on demand.

```text
High Demand
    ↓
More EC2 Instances
```

```text
Low Demand
    ↓
Fewer EC2 Instances
```

---

## 3.2 High Availability

Auto Scaling can distribute EC2 instances across multiple Availability Zones.

```text
Availability Zone 1       Availability Zone 2
       |                         |
      EC2                       EC2
      EC2                       EC2
```

If one Availability Zone has a problem, resources in another Availability Zone can continue serving traffic.

---

## 3.3 Fault Tolerance

Auto Scaling can replace unhealthy instances.

```text
EC2-1 → Healthy
EC2-2 → Healthy
EC2-3 → Unhealthy
```

The unhealthy instance can be replaced:

```text
EC2-3
  ↓
Unhealthy
  ↓
Removed
  ↓
New EC2 launched
  ↓
Healthy
```

---

## 3.4 Cost Efficiency

When demand is low, unnecessary instances can be removed.

For example:

```text
High Traffic:
5 EC2 instances
```

```text
Low Traffic:
2 EC2 instances
```

This can help avoid paying for unnecessary compute capacity.

---

## 3.5 Reduced Manual Work

Without Auto Scaling, administrators may have to manually:

1. Launch EC2 instances.
2. Configure them.
3. Install applications.
4. Configure networking.
5. Register them with a Load Balancer.
6. Monitor them.
7. Remove unnecessary instances.

With Auto Scaling, much of this process can be automated.

---

# 4. Auto Scaling Group (ASG)

An **Auto Scaling Group (ASG)** is a logical group of EC2 instances that AWS manages as a unit.

Example:

```text
             Auto Scaling Group
                    |
        +-----------+-----------+
        |           |           |
        v           v           v
      EC2-1       EC2-2       EC2-3
```

The ASG controls:

* Minimum capacity
* Desired capacity
* Maximum capacity
* Scaling policies
* Health checks
* Availability Zones
* Instance replacement

---

# 5. What Does an Auto Scaling Group Do?

An Auto Scaling Group can perform several important functions.

## Launch Instances

If more capacity is required:

```text
2 EC2
 ↓
3 EC2
 ↓
4 EC2
```

---

## Terminate Instances

If less capacity is required:

```text
4 EC2
 ↓
3 EC2
 ↓
2 EC2
```

---

## Replace Unhealthy Instances

If an instance becomes unhealthy:

```text
EC2-1 → Unhealthy
```

The ASG can replace it:

```text
EC2-1 → Removed
          ↓
      New EC2
          ↓
      Healthy
```

---

## Maintain Desired Capacity

Suppose:

```text
Desired Capacity = 3
```

The Auto Scaling Group attempts to maintain three instances.

If one instance fails:

```text
3 EC2
 ↓
One instance fails
 ↓
2 EC2
 ↓
ASG launches replacement
 ↓
3 EC2
```

---

# 6. Launch Template

A **Launch Template** is a reusable configuration or blueprint used to launch EC2 instances.

It defines how a new EC2 instance should be created.

A Launch Template can contain:

* AMI
* Instance type
* Key pair
* Security Group
* EBS volumes
* IAM instance profile
* User Data
* Network settings
* Monitoring configuration

### Example

```text
Launch Template
       |
       +-- AMI
       +-- Instance Type
       +-- Key Pair
       +-- Security Group
       +-- EBS Storage
       +-- IAM Role
       +-- User Data
```

When the Auto Scaling Group needs a new instance, it can use the Launch Template.

```text
Launch Template
       |
       +---- EC2-1
       |
       +---- EC2-2
       |
       +---- EC2-3
```

---

# 7. Why is a Launch Template Important?

Suppose you manually configure an EC2 instance.

You install:

```text
Apache
Application
Dependencies
Configuration
```

If Auto Scaling launches another EC2 instance, you need that new instance to have the same configuration.

The Launch Template provides the configuration required to create new instances consistently.

For example:

```text
Launch Template
      |
      +-- Amazon Linux
      +-- t3.micro
      +-- Security Group
      +-- IAM Role
      +-- User Data
```

Every new instance launched using that template can start with the defined configuration.

---

# 8. User Data in Launch Template

**User Data** allows commands or scripts to run during instance initialization.

Example:

```bash
#!/bin/bash

dnf install -y httpd
systemctl enable --now httpd

echo "Hello from Auto Scaling" > /var/www/html/index.html
```

This is very useful in Auto Scaling.

When a new EC2 instance is launched:

```text
ASG
 ↓
Launch Template
 ↓
New EC2
 ↓
User Data runs
 ↓
Apache installed
 ↓
Application starts
```

This allows new instances to automatically prepare themselves for production traffic.

---

# 9. Desired Capacity

**Desired capacity** is the number of EC2 instances that the Auto Scaling Group attempts to maintain under normal conditions.

Example:

```text
Minimum = 2
Desired = 3
Maximum = 5
```

The normal target is:

```text
3 EC2 instances
```

Architecture:

```text
Auto Scaling Group
        |
        +-- EC2-1
        +-- EC2-2
        +-- EC2-3
```

If one instance fails:

```text
EC2-1 → Failed

Remaining:
EC2-2
EC2-3
```

The ASG can launch another instance to return toward desired capacity.

---

# 10. Minimum Capacity

**Minimum capacity** defines the smallest number of EC2 instances that the Auto Scaling Group should normally maintain.

Example:

```text
Minimum = 2
```

The ASG should not normally scale below:

```text
2 EC2 instances
```

Example:

```text
Auto Scaling Group
       |
       +-- EC2-1
       +-- EC2-2
```

Minimum capacity helps maintain a baseline level of availability.

---

# 11. Maximum Capacity

**Maximum capacity** defines the largest number of EC2 instances that the Auto Scaling Group can normally launch.

Example:

```text
Maximum = 6
```

The ASG will not normally scale beyond six instances.

```text
EC2-1
EC2-2
EC2-3
EC2-4
EC2-5
EC2-6
```

Maximum capacity helps limit the amount of compute capacity the ASG can create.

---

# 12. Minimum vs Desired vs Maximum

This is one of the most important concepts in Auto Scaling.

Suppose:

```text
Minimum = 2
Desired = 3
Maximum = 5
```

This means:

```text
Minimum
   ↓
2 instances

Desired
   ↓
3 instances normally

Maximum
   ↓
5 instances maximum
```

Visual:

```text
2          3              5
|----------|--------------|
MIN      DESIRED         MAX
```

The Auto Scaling Group can adjust capacity between:

```text
2 → 3 → 4 → 5
```

depending on demand and scaling policies.

---

# 13. Scaling

**Scaling** means increasing or decreasing the number of resources according to workload.

There are two major directions:

1. Scale Out
2. Scale In

---

# 14. Scale Out

**Scale out** means increasing the number of EC2 instances.

Example:

```text
2 EC2
 ↓
3 EC2
 ↓
4 EC2
```

Scale-out normally happens when application demand increases.

Example:

```text
Users increase
     ↓
CPU increases
     ↓
Scaling Policy
     ↓
ASG launches EC2
```

---

# 15. Scale In

**Scale in** means decreasing the number of EC2 instances.

Example:

```text
4 EC2
 ↓
3 EC2
 ↓
2 EC2
```

Scale-in normally happens when application demand decreases.

Example:

```text
Users decrease
     ↓
CPU decreases
     ↓
Scaling Policy
     ↓
ASG removes unnecessary EC2
```

The ASG should respect the configured minimum capacity.

---

# 16. Scaling Policies

A **Scaling Policy** defines how the Auto Scaling Group should adjust its capacity based on conditions or metrics.

Common scaling approaches include:

* Target Tracking Scaling
* Step Scaling
* Simple Scaling
* Scheduled Scaling
* Predictive Scaling

For beginners, **Target Tracking Scaling** is especially important.

---

# 17. Target Tracking Scaling

**Target Tracking Scaling** attempts to maintain a specified target value for a metric.

For example:

```text
Target CPU Utilization = 50%
```

The ASG monitors the metric and adjusts capacity as needed.

### High CPU

```text
CPU = 80%
     ↓
ASG detects high utilization
     ↓
Launch EC2
     ↓
More capacity available
```

### Low CPU

```text
CPU = 20%
     ↓
ASG detects lower utilization
     ↓
May reduce capacity
```

### Simple Definition

> **Target Tracking = Tell Auto Scaling the metric value you want to maintain, and AWS adjusts capacity to try to maintain that target.**

---

# 18. CPU-Based Scaling

One common scaling metric is:

```text
Average CPU Utilization
```

Suppose:

```text
Minimum = 2
Desired = 2
Maximum = 5
Target CPU = 50%
```

Normal state:

```text
EC2-1 → 45%
EC2-2 → 50%
```

The average CPU is close to the target.

No major scaling action may be required.

---

# 19. CPU Scale-Out Example

Suppose application traffic increases.

```text
EC2-1 → 80%
EC2-2 → 85%
```

Average CPU utilization becomes significantly higher than the target.

The scaling policy can cause the ASG to increase capacity.

```text
Before:

EC2-1
EC2-2
```

After:

```text
EC2-1
EC2-2
EC2-3
```

The additional capacity can help handle the increased workload.

---

# 20. CPU Scale-In Example

Suppose application traffic decreases.

```text
CPU
 ↓
40%
 ↓
30%
 ↓
20%
```

The ASG may determine that fewer instances are required.

Example:

```text
3 EC2
 ↓
2 EC2
```

If:

```text
Minimum = 2
```

the ASG should not normally scale below two instances.

---

# 21. Health Checks

Health checks determine whether EC2 instances or Load Balancer targets are healthy.

Example:

```text
ASG
 |
 +-- EC2-1 → Healthy
 |
 +-- EC2-2 → Healthy
 |
 +-- EC2-3 → Unhealthy
```

The Auto Scaling Group can detect the unhealthy instance and replace it.

```text
EC2-3
  ↓
Unhealthy
  ↓
Removed
  ↓
New EC2 launched
  ↓
Health Check
  ↓
Healthy
```

---

# 22. EC2 Health Checks

An Auto Scaling Group can use EC2 health status to determine whether an instance is healthy.

Example:

```text
EC2 Instance
     |
     v
Instance Health
     |
     +---- Healthy
     |
     +---- Unhealthy
```

If an instance is considered unhealthy, the ASG can replace it.

---

# 23. Elastic Load Balancing Health Checks

For an application behind a Load Balancer, the Auto Scaling Group can use **Elastic Load Balancing health checks**.

Example:

```text
                    ALB
                     |
               Target Group
                /         \
               /           \
             EC2-1        EC2-2
               |             |
           Healthy        Unhealthy
                             |
                             v
                           ASG
                             |
                      Replace instance
```

This is useful because an EC2 instance might technically be running while the application itself is not functioning correctly.

For example:

```text
EC2 = Running
Application = Not responding
```

A Load Balancer health check can detect the application-level problem.

---

# 24. Auto Scaling + ALB

Auto Scaling and Load Balancing solve different problems but work very well together.

### Load Balancer

Distributes traffic.

### Auto Scaling Group

Manages EC2 capacity.

Architecture:

```text
                         Users
                           |
                           v
                          ALB
                           |
                    Target Group
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
           EC2-1         EC2-2         EC2-3
             ^             ^             ^
             |             |             |
             +-------------+-------------+
                           |
                  Auto Scaling Group
                           |
                     Launch Template
```

---

# 25. How ALB and Auto Scaling Work Together

Suppose:

```text
Minimum = 2
Desired = 2
Maximum = 5
```

Initially:

```text
ALB
 |
 +---- EC2-1
 +---- EC2-2
```

Traffic increases.

```text
Traffic increases
       ↓
CPU increases
       ↓
Scaling Policy
       ↓
Auto Scaling Group
       ↓
Launch EC2-3
       ↓
Health Check
       ↓
EC2-3 becomes healthy
       ↓
Target Group
       ↓
ALB can send traffic to EC2-3
```

Now:

```text
ALB
 |
 +---- EC2-1
 +---- EC2-2
 +---- EC2-3
```

If traffic increases again:

```text
3 EC2 → 4 EC2
```

and eventually:

```text
4 EC2 → 5 EC2
```

The maximum capacity is five.

---

# 26. Scale-Out Flow

The complete scale-out process can be understood as:

```text
High Traffic
     ↓
CPU increases
     ↓
CloudWatch metric
     ↓
Scaling Policy
     ↓
Auto Scaling Group
     ↓
Launch new EC2
     ↓
Launch Template used
     ↓
EC2 starts
     ↓
Health Check
     ↓
Target becomes Healthy
     ↓
Target Group
     ↓
ALB sends traffic
```

---

# 27. Scale-In Flow

When demand decreases:

```text
Low Traffic
     ↓
CPU decreases
     ↓
CloudWatch metric
     ↓
Scaling Policy
     ↓
Auto Scaling Group
     ↓
Reduce Capacity
     ↓
EC2 instance terminated
     ↓
Traffic continues through remaining healthy targets
```

The ASG respects the configured minimum capacity.

---

# 28. High Availability

**High Availability (HA)** means designing an application so that it can remain available even when some infrastructure fails.

A common AWS architecture is:

```text
                         Users
                           |
                           v
                          ALB
                       /       \
                      /         \
                    AZ-1       AZ-2
                     |           |
                    EC2         EC2
                     \           /
                      \         /
                   Auto Scaling
                      Group
```

The Auto Scaling Group can manage instances across multiple Availability Zones.

Example:

```text
AZ-1
 |
 +-- EC2-1
 +-- EC2-2

AZ-2
 |
 +-- EC2-3
 +-- EC2-4
```

---

# 29. Why Use Multiple Availability Zones?

Suppose all instances are in one Availability Zone:

```text
AZ-1
 |
 +-- EC2-1
 +-- EC2-2
 +-- EC2-3
```

If that Availability Zone experiences a major failure, all application instances could be affected.

Instead:

```text
AZ-1              AZ-2
 |                 |
EC2               EC2
EC2               EC2
```

Now the application has better fault tolerance.

---

# 30. Auto Scaling Across Availability Zones

A common architecture is:

```text
                       ALB
                    /       \
                   /         \
                 AZ-1       AZ-2
                  |           |
                 EC2         EC2
                  |           |
                  +-----+-----+
                        |
                Auto Scaling Group
```

The Auto Scaling Group manages EC2 instances across the selected Availability Zones.

---

# 31. Launch Template vs Auto Scaling Group

These two concepts are commonly confused.

## Launch Template

Answers:

> **How should a new EC2 instance be created?**

Example:

```text
AMI
Instance Type
Security Group
Key Pair
Storage
IAM Role
User Data
```

---

## Auto Scaling Group

Answers:

> **How many EC2 instances should be running and how should capacity change?**

Example:

```text
Minimum
Desired
Maximum
Scaling Policies
Health Checks
Availability Zones
```

### Easy Way to Remember

```text
Launch Template
      ↓
"What should the server look like?"

Auto Scaling Group
      ↓
"How many servers should I run?"
```

---

# 32. Desired, Minimum and Maximum Example

Suppose:

```text
Minimum = 2
Desired = 3
Maximum = 6
```

Normal state:

```text
3 EC2
```

High traffic:

```text
3
 ↓
4
 ↓
5
 ↓
6
```

Low traffic:

```text
6
 ↓
5
 ↓
4
 ↓
3
 ↓
2
```

The ASG should operate within:

```text
2 to 6
```

---

# 33. Scaling Policy Example

Suppose:

```text
Minimum = 2
Desired = 2
Maximum = 6

Target CPU = 50%
```

Traffic increases:

```text
CPU = 80%
```

ASG can scale out:

```text
2 EC2 → 3 EC2
```

If traffic continues increasing:

```text
3 EC2 → 4 EC2
```

Eventually:

```text
6 EC2
```

The maximum capacity has been reached.

---

# 34. Important Point About Scaling

Scaling does not happen instantly.

There are multiple steps:

```text
Metric changes
     ↓
Monitoring detects change
     ↓
Scaling Policy evaluates condition
     ↓
ASG changes desired capacity
     ↓
EC2 launches/terminates
     ↓
Instance initializes
     ↓
Health check
     ↓
Target becomes healthy
     ↓
Traffic can be served
```

The amount of time required depends on:

* Monitoring
* Scaling policy
* Instance startup time
* Application startup time
* Health check configuration

---

# 35. Launch Template and Auto Scaling Relationship

```text
                  Launch Template
                         |
              "How should EC2 look?"
                         |
                         v
                Auto Scaling Group
                         |
              "How many should run?"
                         |
          +--------------+--------------+
          |              |              |
          v              v              v
        EC2-1          EC2-2          EC2-3
```

---

# 36. Auto Scaling + Target Group + ALB

```text
                         Users
                           |
                           v
                          ALB
                           |
                     Listener :80
                           |
                           v
                     Target Group
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
           EC2-1         EC2-2         EC2-3
             ^             ^             ^
             |             |             |
             +-------------+-------------+
                           |
                    Auto Scaling Group
                           |
                    Launch Template
```

---

# 37. Self-Healing Architecture

```text
                       Auto Scaling Group
                              |
                 +------------+------------+
                 |                         |
               EC2-1                    EC2-2
                 |                         |
              Healthy                   Failed
                                           |
                                           v
                                      Health Check
                                           |
                                           v
                                      ASG detects
                                        failure
                                           |
                                           v
                                    Replace instance
                                           |
                                           v
                                         EC2-3
                                           |
                                           v
                                        Healthy
```

---

# 38. Step-by-Step: Create a Launch Template

Now we will create the resources in the AWS Console.

Go to:

```text
AWS Console
    ↓
EC2
    ↓
Launch Templates
```

Click:

```text
Create launch template
```

---

## Step 1 — Launch Template Name

Enter:

```text
clouddevops-launch-template
```

You can provide a description if required.

---

## Step 2 — Choose AMI

Select an appropriate AMI.

For example:

```text
Amazon Linux
```

Use an AMI suitable for your application and architecture.

---

## Step 3 — Choose Instance Type

For a learning lab, choose a small instance type appropriate for your account.

For example:

```text
t3.micro
```

Always verify current AWS pricing and Free Tier eligibility for your account before launching resources.

---

## Step 4 — Key Pair

Select an existing key pair if you need SSH access.

Example:

```text
MyKeyPair
```

If you do not need SSH, follow your lab's security requirements.

---

## Step 5 — Security Group

Select or create an application Security Group.

Example:

```text
EC2-App-SG
```

If the EC2 instances are behind an ALB, the application port should preferably allow traffic from the ALB Security Group.

---

## Step 6 — Storage

Configure the root EBS volume.

For a basic lab, an example is:

```text
8 GiB
gp3
```

The appropriate storage configuration depends on the application.

---

## Step 7 — User Data

User Data can automatically configure the EC2 instance when it launches.

Example:

```bash
#!/bin/bash

dnf install -y httpd
systemctl enable --now httpd

echo "Hello from Auto Scaling" > /var/www/html/index.html
```

Every new EC2 instance launched using this Launch Template can execute the initialization script.

---

## Step 8 — Create Launch Template

Review the configuration.

Click:

```text
Create launch template
```

You now have:

```text
Launch Template
       |
       +-- AMI
       +-- Instance Type
       +-- Security Group
       +-- Storage
       +-- User Data
```

---

# 39. Step-by-Step: Create Auto Scaling Group

Go to:

```text
AWS Console
    ↓
EC2
    ↓
Auto Scaling Groups
```

Click:

```text
Create Auto Scaling group
```

---

# 40. Step 1 — Name the Auto Scaling Group

Enter:

```text
clouddevops-asg
```

---

# 41. Step 2 — Select Launch Template

Select:

```text
clouddevops-launch-template
```

Choose the appropriate template version.

---

# 42. Step 3 — Choose VPC

Select your VPC.

Example:

```text
CloudDevOps-VPC
```

---

# 43. Step 4 — Select Subnets

Select subnets in multiple Availability Zones.

Example:

```text
AZ-1 → Private Subnet 1
AZ-2 → Private Subnet 2
```

For a secure architecture, application EC2 instances can be placed in private subnets while the ALB is placed in public subnets.

---

# 44. Step 5 — Configure Load Balancing

For an application behind an ALB, choose:

```text
Attach to an existing load balancer
```

Select the appropriate Target Group.

Example:

```text
web-target-group
```

Now the relationship becomes:

```text
Auto Scaling Group
        |
        v
Target Group
        |
        v
EC2 Instances
        |
        v
ALB
```

---

# 45. Step 6 — Configure Health Checks

Configure health checks.

For an application behind an ALB, you can use:

```text
ELB health checks
```

This helps the ASG detect instances that are not healthy from the Load Balancer's perspective.

---

# 46. Step 7 — Configure Group Size

For a basic lab, you can use:

```text
Desired capacity: 2
Minimum capacity: 2
Maximum capacity: 4
```

This means:

```text
Minimum = 2
Desired = 2
Maximum = 4
```

The ASG initially attempts to launch two EC2 instances.

---

# 47. Step 8 — Configure Scaling

Select:

```text
Target Tracking Scaling Policy
```

Choose a metric such as:

```text
Average CPU Utilization
```

Example:

```text
Target value = 50%
```

This tells the ASG to adjust capacity to try to maintain average CPU utilization around the specified target.

---

# 48. Step 9 — Create Auto Scaling Group

Review the configuration:

```text
ASG Name:
clouddevops-asg

Launch Template:
clouddevops-launch-template

Minimum:
2

Desired:
2

Maximum:
4

Scaling:
Target Tracking

Metric:
Average CPU Utilization

Target:
50%
```

Click:

```text
Create Auto Scaling group
```

---

# 49. Verify EC2 Instances

Go to:

```text
EC2
 ↓
Instances
```

You should see instances launched by the Auto Scaling Group.

Example:

```text
EC2-1
EC2-2
```

---

# 50. Verify Target Group

Go to:

```text
EC2
 ↓
Target Groups
 ↓
web-target-group
 ↓
Targets
```

You should see:

```text
EC2-1 → Healthy
EC2-2 → Healthy
```

---

# 51. Verify Auto Scaling Group

Go to:

```text
EC2
 ↓
Auto Scaling Groups
 ↓
clouddevops-asg
```

Check:

```text
Desired capacity
Minimum capacity
Maximum capacity
Instances
Scaling policies
Activity history
```

---

# 52. Verify ALB

Go to:

```text
EC2
 ↓
Load Balancers
```

Select your ALB.

Copy its DNS name.

Open the DNS name in your browser.

The ALB should send traffic to healthy EC2 instances managed by the ASG.

---

# 53. Test Auto Scaling

You can test Auto Scaling by generating CPU load on the instances.

Monitor:

```text
CloudWatch
   ↓
EC2 Metrics
   ↓
CPUUtilization
```

If CPU remains high enough for the configured scaling policy to react, the ASG can increase capacity.

Example:

```text
2 EC2
 ↓
High CPU
 ↓
Scaling Policy
 ↓
ASG
 ↓
3 EC2
```

Be careful when generating load because running AWS resources can incur charges.

---

# 54. Test Scale-In

After the CPU test, stop the workload.

CPU should eventually decrease.

If the scaling policy determines that fewer instances are required:

```text
3 EC2
 ↓
Lower CPU
 ↓
Scaling Policy
 ↓
2 EC2
```

The ASG should not normally go below:

```text
Minimum = 2
```

---

# 55. Test Automatic Replacement

Suppose:

```text
EC2-1 → Healthy
EC2-2 → Healthy
```

If an instance becomes unhealthy:

```text
EC2-1
  ↓
Unhealthy
  ↓
Health Check
  ↓
ASG detects failure
  ↓
Instance replaced
  ↓
New EC2 launched
  ↓
Health Check
  ↓
Healthy
```

This demonstrates the self-healing capability of Auto Scaling.

---

# 56. Important Architecture

The architecture you should remember is:

```text
                              USERS
                                |
                                v
                               ALB
                                |
                                v
                         TARGET GROUP
                                |
                +---------------+---------------+
                |               |               |
                v               v               v
              EC2-1           EC2-2           EC2-3
                ^               ^               ^
                |               |               |
                +---------------+---------------+
                                |
                       AUTO SCALING GROUP
                                |
                                v
                        LAUNCH TEMPLATE
```

---

# 57. Complete High-Availability Architecture

A realistic architecture can look like:

```text
                              INTERNET
                                  |
                                  v
                                 ALB
                           /              \
                          /                \
                    Public Subnet      Public Subnet
                       AZ-1                AZ-2
                          |                  |
                          +--------+---------+
                                   |
                              Target Group
                              /           \
                             /             \
                       Private AZ-1      Private AZ-2
                           |                  |
                         EC2-1              EC2-2
                           |                  |
                           +--------+---------+
                                    |
                           Auto Scaling Group
                                    |
                             Launch Template
```

The components have different responsibilities:

* **ALB** distributes application traffic.
* **Target Group** contains backend targets.
* **Auto Scaling Group** manages EC2 capacity.
* **Launch Template** defines how new EC2 instances are created.
* **Scaling Policy** determines when capacity should change.
* **Health Checks** determine whether targets/instances are healthy.
* **Multiple Availability Zones** improve availability and fault tolerance.

---

# 58. Auto Scaling vs Load Balancing

Auto Scaling and Load Balancing are different.

| Feature               | Load Balancer      | Auto Scaling               |
| --------------------- | ------------------ | -------------------------- |
| Main purpose          | Distribute traffic | Manage EC2 capacity        |
| Traffic routing       | Yes                | No                         |
| Launch EC2            | No                 | Yes                        |
| Terminate EC2         | No                 | Yes                        |
| Health checks         | Yes                | Yes                        |
| Scale based on demand | No                 | Yes                        |
| High availability     | Helps              | Helps                      |
| Target Group          | Uses               | Can attach instances to it |

### Easy Way to Remember

> **Load Balancer distributes traffic.**

> **Auto Scaling manages capacity.**

---

# 59. Auto Scaling vs Target Group

## Target Group

Answers:

> **Which backend targets should receive traffic?**

Example:

```text
Target Group
   |
   +-- EC2-1
   +-- EC2-2
   +-- EC2-3
```

---

## Auto Scaling Group

Answers:

> **How many EC2 instances should exist?**

Example:

```text
Auto Scaling Group
   |
   +-- EC2-1
   +-- EC2-2
   +-- EC2-3
```

They can work together:

```text
Auto Scaling Group
       |
       v
EC2 Instances
       |
       v
Target Group
       |
       v
ALB
```

---

# 60. Important Relationships

## Launch Template

```text
Defines how an EC2 instance should be created.
```

## Auto Scaling Group

```text
Manages the number of EC2 instances.
```

## Scaling Policy

```text
Determines when/how capacity should change.
```

## Health Check

```text
Determines whether an instance/target is healthy.
```

## Target Group

```text
Groups backend targets for the Load Balancer.
```

## ALB

```text
Receives and routes application traffic.
```

---

# 61. Interview Questions

## Q1. What is Auto Scaling?

**Answer:**

AWS Auto Scaling automatically adjusts the number of EC2 instances according to application demand and defined scaling policies.

---

## Q2. What is an Auto Scaling Group?

**Answer:**

An Auto Scaling Group is a logical group of EC2 instances that AWS manages to maintain configured minimum, desired, and maximum capacity and replace unhealthy instances.

---

## Q3. What is a Launch Template?

**Answer:**

A Launch Template is a reusable configuration that defines how EC2 instances should be launched, including the AMI, instance type, security groups, storage, IAM role, and user data.

---

## Q4. What is desired capacity?

**Answer:**

Desired capacity is the number of instances that the Auto Scaling Group attempts to maintain under normal conditions.

---

## Q5. What is minimum capacity?

**Answer:**

Minimum capacity is the smallest number of instances that the Auto Scaling Group should normally maintain.

---

## Q6. What is maximum capacity?

**Answer:**

Maximum capacity is the upper limit on the number of instances that the Auto Scaling Group can normally launch.

---

## Q7. What is scale-out?

**Answer:**

Scale-out means increasing the number of EC2 instances when application demand increases.

Example:

```text
2 EC2 → 3 EC2 → 4 EC2
```

---

## Q8. What is scale-in?

**Answer:**

Scale-in means decreasing the number of EC2 instances when application demand decreases.

Example:

```text
4 EC2 → 3 EC2 → 2 EC2
```

---

## Q9. What is Target Tracking?

**Answer:**

Target Tracking is a scaling policy that attempts to maintain a specified target value for a metric.

Example:

```text
Target CPU = 50%
```

---

## Q10. How does CPU-based Auto Scaling work?

**Answer:**

The Auto Scaling Group monitors CPU utilization. When utilization increases and the scaling policy determines that more capacity is required, the ASG can launch additional instances. When utilization decreases and fewer instances are required, it can reduce capacity, subject to the configured minimum and maximum.

---

## Q11. What happens if an EC2 instance becomes unhealthy?

**Answer:**

The Auto Scaling Group can detect the unhealthy instance through its configured health checks and replace it with a new instance to maintain the desired capacity.

---

## Q12. Why use multiple Availability Zones?

**Answer:**

Multiple Availability Zones improve availability and fault tolerance because the application is not dependent on a single Availability Zone.

---

# 62. Important Interview Scenario

### Question:

You have:

```text
Minimum = 2
Desired = 2
Maximum = 5
```

CPU utilization becomes very high.

What happens?

### Answer:

```text
High CPU
   ↓
CloudWatch metric
   ↓
Target Tracking Policy
   ↓
ASG increases desired capacity
   ↓
New EC2 launched using Launch Template
   ↓
Health Check
   ↓
Target becomes healthy
   ↓
Target Group
   ↓
ALB sends traffic to it
```

Eventually:

```text
2 EC2 → 3 EC2 → 4 EC2 → 5 EC2
```

The ASG cannot normally exceed the configured maximum of 5.

---

# 63. Another Interview Scenario

### Question:

One EC2 instance in your Auto Scaling Group fails. What happens?

### Answer:

```text
EC2 fails
   ↓
Health Check
   ↓
ASG detects unhealthy instance
   ↓
Instance removed/replaced
   ↓
Launch Template used
   ↓
New EC2 launched
   ↓
Health Check
   ↓
Healthy
```

The ASG attempts to maintain the desired capacity.

---

# 64. Most Important Concepts to Remember

## 1. Launch Template

```text
How should the EC2 instance be created?
```

## 2. Auto Scaling Group

```text
How many EC2 instances should be running?
```

## 3. Scaling Policy

```text
When should the number of instances change?
```

## 4. Target Tracking

```text
What metric value should Auto Scaling try to maintain?
```

## 5. Health Check

```text
Is the instance/target healthy?
```

## 6. ALB

```text
Where should application traffic be distributed?
```

---

# 65. Quick Revision

```text
AUTO SCALING
     |
     v
AUTO SCALING GROUP
     |
     +---- Minimum Capacity
     |
     +---- Desired Capacity
     |
     +---- Maximum Capacity
     |
     +---- Scaling Policies
     |
     +---- Health Checks
     |
     v
EC2 INSTANCES
     |
     v
TARGET GROUP
     |
     v
ALB
     |
     v
USERS
```

---

# 66. One-Line Definitions

| Topic                  | Definition                                                 |
| ---------------------- | ---------------------------------------------------------- |
| **Auto Scaling**       | Automatically adjusts application capacity based on demand |
| **Auto Scaling Group** | Group that manages EC2 instance capacity                   |
| **Launch Template**    | Blueprint for launching EC2 instances                      |
| **Desired Capacity**   | Number of instances ASG tries to maintain normally         |
| **Minimum Capacity**   | Lowest normal number of instances                          |
| **Maximum Capacity**   | Highest normal number of instances                         |
| **Scale Out**          | Increase number of instances                               |
| **Scale In**           | Decrease number of instances                               |
| **Scaling Policy**     | Defines how/when capacity changes                          |
| **Target Tracking**    | Attempts to maintain a specified target metric             |
| **Health Check**       | Determines whether an instance/target is healthy           |
| **High Availability**  | Designing systems to remain available despite failures     |
| **Availability Zone**  | Isolated location within an AWS Region                     |
| **Target Group**       | Logical group of backend targets                           |
| **ALB**                | Application Load Balancer for HTTP/HTTPS traffic           |

---

# 67. Final Architecture to Memorize

```text
                              USERS
                                |
                                v
                              ALB
                                |
                          LISTENER :80
                                |
                                v
                          TARGET GROUP
                         /     |      \
                        /      |       \
                       v       v        v
                    EC2-1   EC2-2    EC2-3
                      ^       ^        ^
                      |       |        |
                      +-------+--------+
                              |
                     AUTO SCALING GROUP
                              |
                  +-----------+-----------+
                  |                       |
                  v                       v
           SCALING POLICY          HEALTH CHECK
                  |
                  v
             CLOUDWATCH
                  |
                  v
            CPU / METRICS
```

---

# 68. Final Understanding

Do not think of Auto Scaling as simply:

```text
"Automatically create EC2 instances."
```

Think of it as a complete capacity-management system:

```text
                DEMAND
                  |
                  v
               METRICS
                  |
                  v
          SCALING POLICY
                  |
                  v
        AUTO SCALING GROUP
             /         \
            /           \
       SCALE OUT       SCALE IN
          |               |
          v               v
      More EC2        Fewer EC2
          |
          v
   LAUNCH TEMPLATE
          |
          v
   HEALTH CHECK
          |
          v
     TARGET GROUP
          |
          v
         ALB
          |
          v
        USERS
```

The key distinctions are:

> **Launch Template = How to create the server.**

> **Auto Scaling Group = How many servers to maintain.**

> **Desired Capacity = How many servers should normally run.**

> **Minimum Capacity = Lowest number of servers to maintain.**

> **Maximum Capacity = Highest number of servers allowed by the ASG configuration.**

> **Scaling Policy = When/how capacity should change.**

> **Target Tracking = The metric target Auto Scaling tries to maintain.**

> **Health Check = Whether the server/target is healthy.**

> **ALB = Where application traffic is distributed.**

> **Target Group = Which backend targets can receive Load Balancer traffic.**

This is the core foundation of **AWS Auto Scaling + ALB + EC2 High Availability architecture**.

