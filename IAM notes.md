# This file contains my understandings on IAM & identity from a security lens
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

# IAM Note — Trust vs AssumeRole Permission

## Example Scenario

* **Account A** created a role and trusted **Account B** (via account ID)
* **Account B** has two users:

  * `AdminIAMUser` → full admin access
  * `S3ResourceManager` → only S3 permissions

Observed behavior:

* `AdminIAMUser` could **Switch Role** into Account A
* `S3ResourceManager` could **not** switch role

### Why This Happens

Cross-account role assumption requires **two independent approvals**:

1. **Role trust (Account A)**
   The role must trust the external account.

2. **Caller permission (Account B)**
   The user must be explicitly allowed to call `sts:AssumeRole`.

Even if a role trusts an account, **individual users in that account cannot assume the role unless they personally have `sts:AssumeRole` permission**.

### What Passed vs What Failed

* `AdminIAMUser`:

  * Has `AdministratorAccess`
  * Includes `sts:AssumeRole`
  * ✅ Allowed to switch role

* `S3ResourceManager`:

  * Has only `AmazonS3FullAccess`
  * Does NOT include `sts:AssumeRole`
  * ❌ Denied role switch

### Key Insight

> **Trust says “who may assume me”; permissions decide “who actually can.”**

Both must allow the action.

### One-line Takeaway

> **A trusted account is not enough — the user must also be allowed to assume roles.**

---
