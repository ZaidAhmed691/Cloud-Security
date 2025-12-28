# My personal notes and understandings on IAM, Identity & Zero Trust
## Every topic covered and learnt is based on the following 5 questions:
1. What is it?
2. What API or system is involved?
3. What permission allows it?
4. Why does this exist?
5. What can go wrong?

## 1. Creating an IAM User (AWS CLI)

### -- What is it?
Creating a long-lived human identity in AWS using the IAM CreateUser API. This identity can later be granted permissions via IAM policies.

### -- What API or system is involved?
AWS IAM service. The AWS CLI signs and sends the request, which is evaluated by IAM’s policy engine.

### -- What permission allows it?
`iam:CreateUser`. The request succeeds only if allowed and not explicitly denied by any policy.

### -- Why does this exist?
To manage identities in a cloud environment where identity replaces the traditional network perimeter.

### -- What can go wrong?
Over-privileged users, long-lived credentials, and identity sprawl can increase attack surface and enable persistence.

## 2. EC2 Instance Metadata Service (IMDS)

### -- What is it?
IMDS is a local HTTP endpoint that provides instance-specific information and IAM role credentials to workloads running inside an EC2 instance.

### -- What API or system is involved?
EC2 IMDS, which is accessed via the link local IP '169.254.169.254' using HTTP requests from within the instance.

### -- Which permission allows it?
No IAM permission is required to read the instance metadata. Access is controlled by instance configuration (IMDS enabled/IMDSv1/v2 settings)

### -- Why does this exist?
If IMDS is exposed to untrusted workloads or IMDSv1 is enabled, attackers can steal temporary credentials and use them to access AWS resources.

### -- Key Takeaway
IMDS provides identity credentials to EC2 instances not application data. Access to services like S3 always happens through AWS APIs using these credentials.
