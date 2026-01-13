# 1. EC2 IP Addressing Behavior (Public, Private & Elastic IPs)

## Public vs Private IPv4
- **Public IPv4**
  - Used to access EC2 from the **internet**
  - Required for SSH from your local machine
- **Private IPv4**
  - Used for **internal AWS networking**
  - Works only **inside the VPC**
  - Cannot be used to SSH from the public internet

## Connectivity Behavior
- SSH using **public IPv4** → works
- SSH using **private IPv4** from your laptop → fails
- Reason:
  - Your local machine is not inside AWS’s private network
  - Private IPs are not routable over the internet

## Instance Stop vs Reboot
- **Reboot**
  - Public IP stays the same
- **Stop + Start**
  - Public IPv4 **changes**
  - Private IPv4 **remains the same**
- Important when relying on static IP access

## Problem: Public IP Changes
- Stopping and starting an instance causes:
  - Loss of the previous public IPv4
- Breaks:
  - SSH access
  - Hardcoded IP references
  - External integrations

## Solution: Elastic IP (EIP)
- **Elastic IP**
  - Static public IPv4 address
  - Allocated from AWS and **owned by you while allocated**
  - Can be attached to:
    - EC2 instance
    - Network Interface (ENI)
- Public IPv4 stays the same across:
  - Stop / Start
  - Reboots

## Elastic IP Pricing (Important)
- Public IPv4s (including EIP):
  - Charged **~$0.005 per hour**
  - Charged whether used or unused
- Free tier:
  - 750 hours/month of public IPv4
- Best practice:
  - **Release Elastic IPs when not in use**
  - Terminate unused EC2 instances

## Associating an Elastic IP
- Allocate Elastic IP from:
  - EC2 → Elastic IPs
- Associate with:
  - EC2 instance
  - Specific private IP
- After association:
  - Instance public IPv4 = Elastic IP

## Behavior with Elastic IP
- Stop instance:
  - Elastic IP remains attached
- Start instance:
  - Same public IPv4 is reused
- SSH continues to work using the same IP

## Cleanup Best Practices
- Disassociate Elastic IP when no longer needed
- Release Elastic IP to avoid charges
- Terminate EC2 instances after labs

## Exam Takeaways
- Public IP changes on stop/start
- Private IP never changes
- Elastic IP = static public IPv4
- Elastic IPs cost money if left allocated
- Use Elastic IP when a **fixed public IP is required**

---

