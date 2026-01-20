# 13. Auto Scaling Groups (ASG)

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
