# Practice and labs testing

# 1. Create an IAM role to access an AWS resource (console)

### a. Create IAM Role 

IAM → Roles → Create role

Trusted entity: EC2

Permissions: S3 access to the target bucket (bucket-scoped or S3FullAccess for lab)

Role name: LabS3AccessRole

### b. Launch EC2 Instance

EC2 → Launch instance (Amazon Linux / Ubuntu)

Attach IAM role = LabS3AccessRole

Enable Instance Metadata

HttpEndpoint: Enabled

HttpTokens: Optional (or Required if you want IMDSv2 only)

Allow SSH access

### c. Connect to the Instance

SSH using key pair (or Session Manager)

### d. Linux terminal commands (inside EC2)

```bash
aws s3 ls
```

### i. Get EC2 instance meta data

```bash
curl http://169.254.169.254/latest/meta-data
```

### ii. Get IAM and security credentials

```bash
curl http://169.254.169.254/latest/meta-data/iam/security-credentials
```

### iii. Get sts security credentials for an IAM role from the EC2 instance meta data

```bash
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/[put-role-name]
```

# 2. Create an IAM role to access an AWS resource via cross account and switching roles (console)

### a. In Account A (your AWS account)

### i. Create IAM Role

IAM → Roles → Create role

Trusted entity: Another AWS account

Account ID: Account B

Permissions:

Attach S3 Read-Only access
(preferably bucket-scoped, or AmazonS3ReadOnlyAccess for lab)

Role name: ExternalS3ReadOnlyRole

Trust relationship

Ensure trust policy allows Account B to sts:AssumeRole

### b. In Account B (external user account)

### i. Create IAM User

IAM → Users → Create user

Enable AWS Console access (and/or programmatic)

No S3 permissions needed

### ii. Allow user to assume role

Attach policy to the user:

```bash
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Resource": "arn:aws:iam::<ACCOUNT_A_ID>:role/ExternalS3ReadOnlyRole"
    }
  ]
}
```

### iii. Console: Switch Role (Account B user)

Login as IAM user in Account B

Top-right menu → Switch role

Account ID: Account A

Role name: ExternalS3ReadOnlyRole

Display name: optional

You are now temporarily authenticated in Account A

---

# 3. Cross-Account S3 Read/List via External ID

### Goal
Account A allows Account B to get temporary, read-only (List + Get) access to Account A’s S3 resources using STS AssumeRole with an External ID.

### Mental Model

Role lives in Account A

Trust + External ID protect who can assume it

STS issues temporary credentials

Account B uses those credentials to read S3 in Account A

### Actors

Account A → Resource owner (S3 buckets)

Account B → External consumer of data

## a. Part 1 — Account A (Console)

### i. Create IAM Role (Console)

IAM → Roles → Create role

Trusted entity: Another AWS account

Account ID: Account B ID

Require External ID → set value (e.g. ext-s3-read-001)

Role name: ExternalS3ReadListRole

### ii. Attach Permissions

Attach S3 read-only access

Simple lab: AmazonS3ReadOnlyAccess

(or bucket-scoped policy)

Role now:

Trusts Account B

Requires External ID

Has S3 List + Read permissions

## b. Part 2 — Account A (CLI)

Account A does not log in as Account B. Instead, Account A prepares the role ARN + External ID and shares them securely.

(Optional sanity check — Account A identity):

aws sts get-caller-identity

Information shared with Account B:

Role ARN:
```bash
arn:aws:iam::<ACCOUNT_A_ID>:role/ExternalS3ReadListRole
```

External ID:
ext-s3-read-001

⚠️ Account A does NOT generate permanent access keys for Account B

## c. Part 3 — Account B (CLI)

### i. Verify Caller Identity (Account B)

aws sts get-caller-identity

Expected:

Account ID = Account B

### ii. Assume Role in Account A (with External ID)

```bash
aws sts assume-role --role-arn arn:aws:iam::<ACCOUNT_A_ID>:role/ExternalS3ReadListRole --role-session-name --external-id 
```

Output returns temporary credentials:

AccessKeyId

SecretAccessKey

SessionToken

### iii. Set Temporary Credentials (Account B)
```bash
export AWS_ACCESS_KEY_ID=ASIA...
```

```bash
export AWS_SECRET_ACCESS_KEY=...
```

```bash
export AWS_SESSION_TOKEN=...
```

All three are mandatory

### iv. Verify Assumed Role Identity

```bash
aws sts get-caller-identity
```

Expected:

Account ID = Account A

ARN contains: assumed-role/ExternalS3ReadListRole

### v. Part 4 — Validate S3 Access (Account B)

```bash
aws s3 ls
```

```bash
aws s3 ls s3://account-a-bucket
```

```bash
aws s3 cp "s3://account-a-bucket/file name" -
```

### Expected:

✅ List buckets / objects

✅ Read objects

❌ Upload / delete denied

### Security Notes

External ID prevents confused deputy attacks

Role trust + permissions are separate controls

Credentials are temporary and rotatable

---

## 4. Revoking Access for an Assumed Role (Console)

**Goal**
Immediately stop an external account from accessing Account A’s S3 resources by revoking permissions from an IAM role.


### a. Context

* Account B previously assumed a role in Account A
* Role had **S3 List + Read** permissions
* Temporary STS credentials were already issued


### b. Revocation Action (Account A)

1. Go to **IAM → Roles → <RoleName>**
2. **Detach or remove** the S3 read/list permission policy

   * OR delete the role entirely


### c. What Revocation Enforces

* ❌ Any **new** S3 requests using existing STS credentials are denied
* ❌ Future role assumptions inherit the revoked permissions
* ✅ Enforcement is **immediate**


### d. What Revocation Does NOT Do

* Does NOT invalidate already-issued STS credentials
* Does NOT undo data already accessed or downloaded
* Does NOT wait for session expiration

### e. Verification (Account B CLI)

```bash
aws s3 ls s3://account-a-bucket
```

Expected result:

* `AccessDenied`

### Key Security Insight

> STS credentials are evaluated **at request time**, not at issuance time.

Changing a role’s policy instantly changes what existing sessions can do.

---

## 5. Constrained Admin with Permission Boundary (James)

**Goal**

Create an IAM test user **James** with full admin-style capabilities **without allowing privilege escalation**, self-permission changes, or removal of guardrails.

## High-level Idea

> James can manage AWS resources and IAM for others,
> but **cannot change his own permissions or bypass boundaries**.

This is achieved using:

* `AdministratorAccess` (identity policy)
* A **permission boundary** (maximum permissions)

## Lab Flow

### a. Create Permission Boundary Policy

* IAM → Policies → Create policy
* Type: **Customer managed policy**
* Purpose: restrict IAM privilege escalation and self-management

(Policy JSON documented separately in policy notes)

### b. Create IAM User: James

* IAM → Users → Create user
* Username: `James`
* Enable console and/or programmatic access

### c. Attach Identity Policy

* Attach **AdministratorAccess** managed policy

This grants James broad admin capabilities.

### d. Apply Permission Boundary

* Set **permission boundary** to the custom boundary policy

This caps what James can actually do, even with admin access.

### e. Validate Behavior (Mental Checks)

James **can**:

* Manage AWS services (EC2, S3, VPC, CloudWatch, etc.)
* Manage IAM users and groups **other than himself**

James **cannot**:

* Change his own permissions
* Remove permission boundaries
* Modify boundary policies
* Escalate privileges

### Security Outcome

* Least privilege enforced at the admin level
* No self-escalation paths
* Safe delegation of IAM responsibilities

### One-line Takeaway

> **Permission boundaries turn full admins into safe admins.**

### One-line Takeaway

**Revoking permissions stops future access immediately by changing authorization, not credentials.**

---
