# 1. Auto Scaling Groups (ASG)

## What is an Auto Scaling Group?
An Auto Scaling Group (ASG) automatically adjusts the number of EC2 instances running your application based on demand.

Traffic load can change over time, and instead of manually launching or terminating EC2 instances, ASG automates this process.

## Core Goals of ASG
- **Scale out** → add EC2 instances when load increases  
- **Scale in** → remove EC2 instances when load decreases  

The size of an ASG changes dynamically over time.

## Capacity Settings
Every ASG defines three key values:

- **Minimum capacity**  
  Lowest number of instances that must always be running  
  Example: 2

- **Desired capacity**  
  Target number of instances the ASG tries to maintain  
  Example: 4

- **Maximum capacity**  
  Upper limit of instances allowed  
  Example: 7

ASG scales **between min and max**, always trying to reach the desired capacity.

## ASG + Load Balancer (Very Important)
When an ASG is attached to a load balancer:

- New instances are **automatically registered**
- Terminated instances are **automatically deregistered**
- Traffic is evenly distributed across healthy instances

Health checks from the load balancer can be passed to the ASG.

If an instance is:
- Marked **unhealthy**
- Fails health checks  

→ ASG **terminates it and replaces it automatically**

## Self-Healing Capability
One of the biggest advantages of ASG:

- Unhealthy instance → terminated
- New instance → launched automatically
- No manual intervention required

This ensures high availability.

## Cost Model
- **ASG itself is free**
- You only pay for:
  - EC2 instances
  - EBS volumes
  - Load balancers
  - Other attached resources

## Launch Templates
ASGs require a **Launch Template**.

Launch Configurations are deprecated.

A Launch Template defines **how instances are created**, including:
- AMI
- Instance type
- User data
- EBS volumes
- Security groups
- Key pair
- IAM role
- Network & subnets
- Load balancer attachment

Essentially, it’s a reusable EC2 configuration blueprint.

## Scaling Policies & CloudWatch
ASG integrates with **CloudWatch alarms** to automate scaling.

Example:
- Metric: Average CPU utilization
- Condition: CPU > 70%
- Action: Scale out (add instances)

CloudWatch alarms can trigger:
- **Scale out policies** → increase capacity
- **Scale in policies** → decrease capacity

This is what makes ASG truly *automatic*.

## Exam Takeaways
- ASG automatically adds/removes EC2 instances
- Scale out = add instances
- Scale in = remove instances
- Works best with Load Balancers
- Replaces unhealthy instances automatically
- Uses Launch Templates
- Scaling driven by CloudWatch alarms

---

# 2. Auto Scaling Group (ASG) – Hands-On Practice

## Prerequisites
- Terminate **all existing EC2 instances**
- Ensure **zero instances** are running before starting

## Step 1: Create an Auto Scaling Group
- Navigate to **Auto Scaling Groups**
- Click **Create Auto Scaling Group**
- Name: `Demo-ASG`

## Step 2: Create a Launch Template
A launch template defines **how EC2 instances are created** inside the ASG.

### Launch Template Settings
- **Name:** `my-demo-template`
- **AMI:** Amazon Linux 2 (x86, Free Tier eligible)
- **Instance type:** `t2.micro`
- **Key pair:** `ec2-tutorial`
- **Security group:** `launch-wizard-1`
- **Storage:** 8 GB gp2
- **User data:**  
  - Same script used previously to install and start a web server
  - Ensures every EC2 instance serves HTTP traffic

Create the launch template and select **Version 1**.

## Step 3: Configure Instance Launch Options
- **Instance type override:** Reset to launch template
- **VPC:** Default
- **Availability Zones:** Select multiple AZs
- **AZ distribution:** Balanced best effort

This spreads instances across AZs automatically.

## Step 4: Attach Load Balancer
- Attach to an **existing load balancer**
- Select target group: `demo-tg-alb`
- Enable:
  - **EC2 health checks**
  - **Load balancer health checks**

This allows ASG to replace unhealthy instances.

## Step 5: Set Capacity
- **Minimum capacity:** 1
- **Desired capacity:** 1
- **Maximum capacity:** 1

(No automatic scaling policies yet)

## Step 6: Create ASG
- Review configuration
- Click **Create Auto Scaling Group**

## Step 7: Verify Instance Creation
- Go to **Activity History**
  - Observe instance launch activity
- Go to **Instance Management**
  - One EC2 instance should be running
- Check **Target Group**
  - Instance registers automatically
  - Initially unhealthy → becomes healthy after boot

Once healthy:
- Load Balancer returns `Hello World`

## Step 8: Scale Out (Manual)
- Edit ASG
- Update:
  - **Desired capacity:** 2
  - **Maximum capacity:** 2
- Save changes

### Result
- ASG launches a second EC2 instance
- Instance registers with target group
- Load balancer distributes traffic between both instances

## Step 9: Verify Load Balancing
- Refresh ALB DNS name
- Responses alternate between **two different instance IPs**

## Step 10: Scale In (Manual)
- Edit ASG
- Set:
  - **Desired capacity:** 1
- Save changes

### Result
- ASG terminates one instance
- Instance is deregistered from target group
- One EC2 instance remains active

## Key Observations
- ASG automatically:
  - Launches instances to meet desired capacity
  - Terminates instances when scaling in
  - Registers/deregisters instances with the load balancer
- Health checks determine replacement behavior
- All activity is visible in **Activity History**

## Common Troubleshooting
If instances keep terminating:
- Check **security groups**
- Verify **user data script**
- Ensure health check path is reachable

## Summary
- ASG + ALB = automated, self-healing infrastructure
- Manual scaling demonstrates core ASG behavior
- Automatic scaling policies build on this foundation

---

# 3. Auto Scaling Group – Scaling Policies

Auto Scaling Groups (ASGs) support **multiple scaling strategies** depending on how predictable and dynamic your workload is.

---

## Dynamic Scaling

Dynamic scaling reacts **automatically** to changes in load.

### 1. Target Tracking Scaling
- **Simplest to configure**
- You define:
  - A **metric** (e.g. CPU Utilization)
  - A **target value** (e.g. 40%)
- The ASG automatically:
  - **Scales out** when metric goes above target
  - **Scales in** when metric goes below target

**Example:**
- Metric: Average CPU Utilization
- Target: 40%
- ASG keeps CPU close to 40% by adding or removing instances

### 2. Simple / Step Scaling
- Based on **CloudWatch alarms**
- You explicitly define:
  - When to **add capacity**
  - When to **remove capacity**

**Example:**
- If CPU > 70% → add 2 instances
- If CPU < 20% → remove 1 instance

Step scaling allows more granular control than target tracking.

## Scheduled Scaling

Used when traffic patterns are **known in advance**.

**Example:**
- Every Friday at 5:00 PM:
  - Increase minimum capacity to 10
- Every Saturday at 2:00 AM:
  - Reduce minimum capacity back to 2

This is useful for:
- Business hours
- Marketing campaigns
- Batch jobs

## Predictive Scaling

- Uses **machine learning**
- Continuously analyzes **historical load**
- Forecasts future demand
- Schedules scaling actions **ahead of time**

Best for:
- Cyclical traffic
- Repeating usage patterns
- Long-running applications

## Common Scaling Metrics

Choosing the right metric depends on your application.

### CPU Utilization
- Most common
- Good when requests are compute-heavy
- Higher CPU = more load → scale out

### RequestCountPerTarget (ALB)
- Application-specific
- Based on known optimal request handling

**Example:**
- Each instance handles ~1000 requests optimally
- Scale when requests per target exceed threshold

### Network In / Out
- Useful for:
  - Upload-heavy applications
  - Download-heavy workloads
- Scale when network bandwidth becomes the bottleneck

### Custom CloudWatch Metrics
- Application-specific metrics
- Pushed directly to CloudWatch
- Allows very precise scaling behavior

## Scaling Cooldown

After any scaling activity:
- ASG enters a **cooldown period**
- Default: **300 seconds (5 minutes)**

During cooldown:
- No new scaling actions occur
- Allows:
  - Metrics to stabilize
  - New instances to become effective

### Why Cooldown Matters
Prevents:
- Rapid scale-out / scale-in loops
- Over-reaction to temporary spikes

## Best Practices

- Use **pre-baked AMIs**
  - Faster boot time
  - Faster scaling response
- Reduce cooldown if instances become ready quickly
- Enable **detailed monitoring**
  - Metrics every 1 minute
  - Faster reaction time

## Summary

- **Target Tracking** → simplest, recommended
- **Step Scaling** → fine-grained control
- **Scheduled Scaling** → predictable traffic
- **Predictive Scaling** → recurring patterns
- Cooldown prevents instability
- Metrics choice depends on workload

---
