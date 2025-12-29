# Practice and labs testing

# 1. Create an IAM role to access an AWS service (console)

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
