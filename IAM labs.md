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

aws s3 ls
### i. Get EC2 instance meta data
curl http://169.254.169.254/latest/meta-data
### ii. Get IAM and security credentials
curl http://169.254.169.254/latest/meta-data/iam/security-credentials
### iii. Get sts security credentials for an IAM role from the EC2 instance meta data
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/[put-role-name]

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

### iii. Console: Switch Role (Account B user)

Login as IAM user in Account B

Top-right menu → Switch role

Account ID: Account A

Role name: ExternalS3ReadOnlyRole

Display name: optional

You are now temporarily authenticated in Account A

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
arn:aws:iam::<ACCOUNT_A_ID>:role/ExternalS3ReadListRole

External ID:
ext-s3-read-001

⚠️ Account A does NOT generate permanent access keys for Account B

## c. Part 3 — Account B (CLI)

### i. Verify Caller Identity (Account B)

aws sts get-caller-identity

Expected:

Account ID = Account B

### ii. Assume Role in Account A (with External ID)
aws sts assume-role \
  --role-arn arn:aws:iam::<ACCOUNT_A_ID>:role/ExternalS3ReadListRole
  --role-session-name 
  --external-id 

Output returns temporary credentials:

AccessKeyId

SecretAccessKey

SessionToken

### iii. Set Temporary Credentials (Account B)
export AWS_ACCESS_KEY_ID=ASIA...
export AWS_SECRET_ACCESS_KEY=...
export AWS_SESSION_TOKEN=...

All three are mandatory

### iv. Verify Assumed Role Identity

aws sts get-caller-identity

Expected:

Account ID = Account A

ARN contains: assumed-role/ExternalS3ReadListRole

### v. Part 4 — Validate S3 Access (Account B)
aws s3 ls
aws s3 ls s3://account-a-bucket
aws s3 cp s3://account-a-bucket/file.txt -

Expected:

✅ List buckets / objects

✅ Read objects

❌ Upload / delete denied

### Security Notes

External ID prevents confused deputy attacks

Role trust + permissions are separate controls

Credentials are temporary and rotatable
