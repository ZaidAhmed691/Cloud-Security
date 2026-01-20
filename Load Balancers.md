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

# 3. ALB Hands-On with EC2 Target Group

## Launch Backend EC2 Instances
- Launch 2 Amazon Linux 2 EC2 instances (t2.micro)
- Use existing security group allowing:
  - HTTP (80)
  - SSH (not used here but already allowed)
- Add user data to install Apache and serve a simple Hello World page
- Rename instances:
  - My First Instance
  - My Second Instance
- Verify each instance via its public IPv4 → both return Hello World

## Create Application Load Balancer
- EC2 → Load Balancers → Create
- Choose **Application Load Balancer**
- Name: DemoALB
- Scheme: Internet-facing
- IP type: IPv4
- Deploy across all available AZs
- Create new SG for ALB:
  - Allow inbound HTTP from anywhere (0.0.0.0/0)
  - Remove default SG and attach only ALB SG

## Create Target Group
- Name: demo-tg-alb
- Target type: Instances
- Protocol: HTTP / Port 80
- Health check path: default (/)
- Register both EC2 instances on port 80

## Attach Target Group to ALB Listener
- Listener: HTTP : 80
- Forward traffic to demo-tg-alb
- Create Load Balancer

## Verify Load Balancing
- Open ALB DNS name in browser
- Refresh page repeatedly → traffic alternates between instances
- Confirms round-robin behavior

## Health Check Demonstration
- Stop one EC2 instance
- Target Group marks it **Unhealthy / Unused**
- ALB automatically routes only to healthy instance
- Restart stopped instance
- Health checks pass → ALB resumes sending traffic to both

## Exam Takeaways
- ALB routes via Target Groups
- Health checks control traffic flow automatically
- Users only see ALB DNS, never backend IPs
- Backend SG must allow traffic from ALB SG only
- Load balancers improve HA and fault tolerance

---

# 4. ALB Advanced Concepts (Security + Listener Rules)

## Restrict EC2 Access to Only the Load Balancer
- Previously EC2 instances allowed HTTP from anywhere
- Updated EC2 Security Group inbound rule:
  - Removed 0.0.0.0/0 HTTP rule
  - Allowed HTTP only from ALB Security Group
- Result:
  - Direct access via EC2 Public IP fails (timeout)
  - ALB still successfully forwards traffic
- Purpose:
  - Force all users to go through the Load Balancer
  - Improves security and control

## ALB Listener Rules
- ALB → Listeners → View/Edit Rules
- Default rule:
  - All traffic → demo target group
- Added custom rule:
  - Condition: Path = `/error`
  - Action: Return fixed response
  - Response:
    - Status: 404
    - Message: "Not Found – Custom Error"
  - Priority set to 5 (higher than default)

## Rule Behavior
- Visiting ALB DNS normally → routes to EC2
- Visiting ALB DNS + `/error` → returns custom 404 page
- ALB evaluates rules by priority (lower number = higher priority)

## Exam Takeaways
- Best practice: EC2 should only allow traffic from ALB SG
- Listener rules allow routing or fixed responses
- Path-based routing is a core ALB feature
- Rules are processed in priority order

---

# 5. Network Load Balancer (NLB)

## Overview
- Layer 4 Load Balancer
- Supports:
  - TCP
  - UDP
  - TLS over TCP
- Used for extreme performance workloads
- Handles millions of requests per second
- Ultra-low latency

## Static IP Feature
- NLB provides **one static IP per Availability Zone**
- Can associate an Elastic IP per AZ
- Useful when:
  - Clients must whitelist fixed IPs
  - Regulatory/network restrictions require static endpoints
- Exam keyword: **Static IP → NLB**

## How NLB Routes Traffic
- Uses Target Groups like ALB
- Traffic forwarded at TCP/UDP level (no HTTP awareness)

## Target Groups Supported
- EC2 Instances
- Private IP Addresses
  - Can include:
    - EC2 private IPs
    - On-prem servers (via VPN/Direct Connect)
- Can place NLB **in front of an ALB**
  - NLB = static IP entry point
  - ALB = HTTP routing intelligence

## Health Checks
NLB supports:
- TCP
- HTTP
- HTTPS

(If backend app exposes HTTP/HTTPS health endpoints, NLB can use them)

## Exam Takeaways
- UDP traffic always uses NLB
- Need ultra-high performance → NLB
- Need static IP for LB → NLB
- Can combine NLB + ALB for static IP + HTTP routing

---

# 6. Network Load Balancer — Hands-On Summary

## Goal
Create an internet-facing NLB and route traffic to two EC2 instances.

## Steps Performed

### 1. Create Network Load Balancer
- Name: DemoNLB  
- Scheme: Internet-facing  
- IP type: IPv4  
- Enabled all AZs → AWS assigned **static IP per AZ**  
- Attached Security Group: demo-sg-nlb  
  - Allowed inbound TCP 80 from anywhere  

### 2. Create Target Group (for NLB)
- Target type: Instances  
- Name: demo-tg-nlb  
- Protocol: TCP  
- Port: 80  
- Health check protocol: HTTP  
- Registered both EC2 instances  

### 3. Attach Target Group to Listener
- Listener: TCP : 80  
- Forward to demo-tg-nlb  

---

## Issue Faced
Targets showed **Unhealthy**

### Root Cause
EC2 Security Group only allowed traffic from ALB SG, not from NLB.

### Fix Applied
Updated EC2 Security Group:
- Added inbound HTTP rule  
- Source = demo-sg-nlb (NLB Security Group)

Result → Targets became **Healthy**

---

## Validation
- Accessed NLB DNS  
- Refreshed page multiple times  
- Response alternated between instance IPs → confirms load balancing  

---

## Cleanup
- Deleted DemoNLB  
- Deleted demo-tg-nlb  
- Optional: removed demo-sg-nlb  

---

## Exam Takeaways
- NLB requires EC2 SG to explicitly allow traffic from NLB SG  
- Health checks fail if backend SG blocks LB  
- NLB works at TCP level (no HTTP routing awareness)  
- Each AZ provides static IP automatically  

---

# 7. Gateway Load Balancer (GWLB)

## What it is
- Newest AWS load balancer type  
- Designed to deploy, scale, and manage **third-party network appliances**  
- Works as both:
  - A transparent network gateway
  - A load balancer for security appliances  

## Why it exists
Used when **all network traffic must be inspected** before reaching applications, such as:

- Firewalls  
- Intrusion Detection / Prevention Systems (IDS/IPS)  
- Deep Packet Inspection  
- Traffic filtering or modification  

## How it works (Concept)

Normal flow without GWLB:
- User → ALB → EC2

With GWLB:
- User traffic → Gateway Load Balancer → Security Appliances → Back to GWLB → Application

Key idea:
- Traffic is **forced through appliances** using VPC route table updates
- Appliances inspect traffic and:
  - Forward back if allowed
  - Drop if blocked  

Fully transparent to backend apps.

## Technical Layer
- Operates at **Layer 3 (IP level)**  
- Uses **GENEVE protocol on port 6081**  

Exam trigger:
> If you see GENEVE → think GWLB immediately.

## Target Groups for GWLB
Can register:
- EC2 instances (security appliances)
- Private IP addresses (including on-prem appliances)

## Key Exam Takeaways
- Used for centralized traffic inspection
- Combines routing + load balancing for appliances
- Layer 3 only
- Requires route table changes
- Protocol = GENEVE (6081)
- No hands-on expected in exam, concept only  

---

# 8. Sticky Sessions (Session Affinity)

## What Sticky Sessions Are
- Ensures a client is always routed to the **same backend instance**
- Achieved using cookies managed by the load balancer or the application
- Normally, load balancers distribute each request independently across targets  
- With stickiness enabled → repeated requests from a client go to the same instance

## Why Use Sticky Sessions
- Preserves session data (login state, carts, user context)
- Important for stateful applications

⚠ Downside:
- Can create uneven load if some users remain tied to one instance for long periods

## Supported Load Balancers
- Classic Load Balancer (CLB)
- Application Load Balancer (ALB)
- Network Load Balancer (NLB)

## How Stickiness Works
- Load balancer sends a cookie to the client
- Client includes cookie in future requests
- LB reads cookie and routes request to the same instance
- When cookie expires → client may be routed elsewhere

## Cookie Types

### 1) Duration-Based Cookies (LB generated)
- Managed fully by the Load Balancer
- ALB cookie name: `AWSALB`
- CLB cookie name: `AWSELB`
- Expiration defined at target group level (1 second → 7 days)

### 2) Application-Based Cookies
- Cookie created by the backend application
- You define the cookie name per target group
- Cannot use reserved names:
  - AWSALB  
  - AWSALBAPP  
  - AWSALBTG  

Load balancer still tracks and enforces affinity using that custom cookie.

## Demo Outcome (Concept)
- Before enabling stickiness → refresh cycles across instances
- After enabling → repeated refresh returns same backend instance
- Cookie visible in browser dev tools under request/response headers

## Exam Takeaways
- Sticky sessions = session affinity
- Implemented via cookies
- Used for session persistence
- Works with CLB, ALB, NLB
- May cause imbalance across instances
- Two cookie methods: duration-based & application-based

---

# 9. Cross-Zone Load Balancing

## What It Means
Cross-zone load balancing allows each load balancer node to distribute traffic across **all registered instances in all Availability Zones**, instead of only routing to targets within its own AZ.

## Scenario With Cross-Zone Enabled
- 2 AZs  
- AZ-A → 2 EC2 instances  
- AZ-B → 8 EC2 instances  
- Total instances = 10  

Traffic flow:
- Client sends 50% to LB node in AZ-A  
- Client sends 50% to LB node in AZ-B  
- EACH LB node distributes traffic evenly across ALL 10 instances  
- Every instance receives ~10% total traffic  

Result:
✔ Even distribution  
✔ No AZ imbalance impact  

## Scenario Without Cross-Zone
Traffic is restricted per AZ.

Traffic flow:
- Client sends 50% traffic to AZ-A LB → split between 2 instances → 25% each  
- Client sends 50% traffic to AZ-B LB → split between 8 instances → ~6.25% each  

Result:
❌ Instances in AZ-A overloaded  
❌ Uneven utilization across AZs  

## Default Behavior Per Load Balancer

### Application Load Balancer (ALB)
- Cross-zone: **Enabled by default**
- No inter-AZ data charges
- Can only disable at Target Group level

### Network Load Balancer (NLB)
- Cross-zone: Disabled by default
- Enabling causes inter-AZ data charges

### Gateway Load Balancer (GWLB)
- Cross-zone: Disabled by default
- Enabling causes inter-AZ data charges

### Classic Load Balancer (CLB)
- Disabled by default
- Enabling does NOT incur inter-AZ charges

## Key Exam Takeaways
- Cross-zone = distributes across ALL AZ targets  
- Prevents imbalance when AZs have unequal instance counts  
- ALB always ON by default  
- NLB/GWLB OFF by default + charges when enabled  
- CLB OFF by default but no charge when enabled  

---
