# Practice and labs testing

## 1. Create an IAM role to access an AWS service (console)

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
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/[put_role_name]
