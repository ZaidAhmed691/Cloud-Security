# My personal notes and understandings on IAM, Identity & Zero Trust
## Every topic covered and learnt is based on the following 5 questions:
1. What is it?
2. What API or system is involved?
3. What permission allows it?
4. Why does this exist?
5. What can go wrong?

## Creating an IAM User (AWS CLI)

### 1. What is it?
Creating a long-lived human identity in AWS using the IAM CreateUser API. This identity can later be granted permissions via IAM policies.

### 2. What API or system is involved?
AWS IAM service. The AWS CLI signs and sends the request, which is evaluated by IAM’s policy engine.

### 3. What permission allows it?
`iam:CreateUser`. The request succeeds only if allowed and not explicitly denied by any policy.

### 4. Why does this exist?
To manage identities in a cloud environment where identity replaces the traditional network perimeter.

### 5. What can go wrong?
Over-privileged users, long-lived credentials, and identity sprawl can increase attack surface and enable persistence.


