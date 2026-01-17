# 1. Load Balancing in AWS (ELB)

## What is Load Balancing
- A Load Balancer distributes incoming traffic across multiple backend EC2 instances.
- Users connect to **one endpoint only** (the Load Balancer), not directly to instances.
- Traffic is evenly spread to avoid overloading any single server.

## Why Use a Load Balancer
- Single point of access for applications  
- Automatically reroutes traffic away from failed instances  
- Built-in health checks  
- SSL termination (handles HTTPS)  
- Cookie stickiness (session persistence)  
- High availability across AZs  
- Separation of public vs private traffic  
- Deep integration with AWS ecosystem  

## ELB is Fully Managed
- AWS handles scaling, patching, maintenance  
- No infrastructure to manage  
- Cheaper and far more scalable than self-managed solutions  
- Integrates with:
  - EC2
  - Auto Scaling Groups
  - ECS
  - ACM (certificates)
  - CloudWatch
  - Route 53
  - WAF
  - Global Accelerator

## Health Checks
- ELB checks instance health before routing traffic.
- Configured using:
  - Protocol (HTTP/TCP)
  - Port
  - Path (e.g., `/health`)
- Instance must return HTTP **200 OK** to be considered healthy.
- Unhealthy instances receive **no traffic**.

## Types of AWS Load Balancers

### Classic Load Balancer (CLB — Legacy)
- Old generation (2009)
- Supports HTTP, HTTPS, TCP, SSL
- Deprecated — avoid for new deployments.

### Application Load Balancer (ALB)
- Layer 7 (HTTP/HTTPS/WebSockets)
- Advanced routing:
  - Path-based
  - Host-based
- Best for web applications & microservices.

### Network Load Balancer (NLB)
- Layer 4 (TCP, TLS, UDP)
- Extremely high performance
- Ultra-low latency
- Static IP support.

### Gateway Load Balancer (GWLB)
- Layer 3 (IP level)
- Used for security appliances:
  - Firewalls
  - Intrusion detection
  - Deep packet inspection.

## Internal vs External LB
- **Internet-facing** → Public apps  
- **Internal** → Private apps inside VPC

## Security Groups with ELB

### Load Balancer SG
- Allows traffic from anywhere  
  - Port 80 / 443  
  - Source: `0.0.0.0/0`

### EC2 SG (Important Exam Concept)
- EC2 only accepts traffic **from the Load Balancer SG**
- Source = Security Group ID, not IP range.
- This ensures instances are protected from direct public access.

## Key Exam Takeaways
- ELB always sits between users and EC2 instances  
- Health checks determine traffic routing  
- CLB = legacy  
- ALB = HTTP smart routing  
- NLB = high performance Layer 4  
- EC2 SG should reference LB SG (not open to world)

---

# 2. Application Load Balancer (ALB)

## Overview
- Layer 7 Load Balancer (HTTP/HTTPS only)
- Routes traffic to **Target Groups**
- Supports microservices and container-based apps
- One ALB can serve multiple applications

## Key Features
- HTTP/2 and WebSocket support
- Redirect HTTP → HTTPS
- Advanced routing:
  - Path-based (`/users`, `/search`)
  - Host-based (`api.example.com`)
  - Query string-based (`?Platform=Mobile`)
  - Header-based routing

## Why ALB Over Classic LB
- Classic LB = 1 LB per app
- ALB = many apps behind a single LB
- Better for modern architectures

## Target Groups
ALB can route traffic to:
- EC2 instances
- Auto Scaling Groups
- ECS tasks (containers)
- Lambda functions
- Private IP addresses (on-prem or inside VPC)

Health checks happen **per target group**, not per instance directly.

## Example Architecture
- 1 ALB
- Target Group A → `/users`
- Target Group B → `/search`
- ALB rules determine routing based on URL path

## Hybrid Routing Example
- TG1 → AWS EC2
- TG2 → On-prem private servers
- ALB rule:
  - `?Platform=Mobile` → TG1
  - `?Platform=Desktop` → TG2

## ALB Networking Behavior
- Clients connect to ALB public endpoint
- ALB terminates connection
- ALB forwards request to EC2 using private IP

Backend instances **do not see real client IP** directly.

Client IP passed via headers:
- `X-Forwarded-For`
- `X-Forwarded-Port`
- `X-Forwarded-Proto`

## Exam Takeaways
- ALB = Layer 7 only
- Uses Target Groups
- Supports advanced routing rules
- Can route to Lambda + IP targets
- Client IP preserved only via X-Forwarded headers
- Best LB for microservices and containers

---
