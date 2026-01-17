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

---

## Health Checks
- ELB checks instance health before routing traffic.
- Configured using:
  - Protocol (HTTP/TCP)
  - Port
  - Path (e.g., `/health`)
- Instance must return HTTP **200 OK** to be considered healthy.
- Unhealthy instances receive **no traffic**.

---

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

---

## Internal vs External LB
- **Internet-facing** → Public apps  
- **Internal** → Private apps inside VPC

---

## Security Groups with ELB

### Load Balancer SG
- Allows traffic from anywhere  
  - Port 80 / 443  
  - Source: `0.0.0.0/0`

### EC2 SG (Important Exam Concept)
- EC2 only accepts traffic **from the Load Balancer SG**
- Source = Security Group ID, not IP range.
- This ensures instances are protected from direct public access.

---

## Key Exam Takeaways
- ELB always sits between users and EC2 instances  
- Health checks determine traffic routing  
- CLB = legacy  
- ALB = HTTP smart routing  
- NLB = high performance Layer 4  
- EC2 SG should reference LB SG (not open to world)
