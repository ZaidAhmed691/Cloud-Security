# JSON Policies

# 1. Permission Boundary for Constrained Admin (James)

**Purpose**

Define a permission boundary that allows broad IAM and CloudWatch administration **while preventing self-permission changes, boundary removal, and privilege escalation**.

## Permission Boundary Policy (JSON)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "CreateOrChangeOnlyWithBoundary",
      "Effect": "Allow",
      "Action": [
        "iam:PutUserPermissionsBoundary",
        "iam:PutUserPolicy",
        "iam:DeleteUserPolicy",
        "iam:AttachUserPolicy",
        "iam:DetachUserPolicy",
        "iam:CreateUser"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "iam:PermissionsBoundary": "arn:aws:iam::<ACCOUNT_ID>:policy/userboundary"
        }
      }
    },
    {
      "Sid": "CloudWatchAndOtherIAMTasks",
      "Effect": "Allow",
      "Action": [
        "cloudwatch:*",
        "iam:GetUser",
        "iam:ListUsers",
        "iam:DeleteUser",
        "iam:UpdateUser",
        "iam:CreateAccessKey",
        "iam:CreateLoginProfile",
        "iam:GetAccountPasswordPolicy",
        "iam:GetLoginProfile",
        "iam:ListGroups",
        "iam:ListGroupsForUser",
        "iam:CreateGroup",
        "iam:GetGroup",
        "iam:DeleteGroup",
        "iam:UpdateGroup",
        "iam:CreatePolicy",
        "iam:DeletePolicy",
        "iam:DeletePolicyVersion",
        "iam:GetPolicy",
        "iam:GetPolicyVersion",
        "iam:GetUserPolicy",
        "iam:GetRolePolicy",
        "iam:ListPolicies",
        "iam:ListPolicyVersions",
        "iam:ListEntitiesForPolicy",
        "iam:ListUserPolicies",
        "iam:ListAttachedUserPolicies",
        "iam:ListRolePolicies",
        "iam:ListAttachedRolePolicies",
        "iam:SetDefaultPolicyVersion",
        "iam:SimulatePrincipalPolicy",
        "iam:SimulateCustomPolicy"
      ],
      "NotResource": "arn:aws:iam::<ACCOUNT_ID>:user/James"
    },
    {
      "Sid": "NoBoundaryPolicyEdit",
      "Effect": "Deny",
      "Action": [
        "iam:CreatePolicyVersion",
        "iam:DeletePolicy",
        "iam:DeletePolicyVersion",
        "iam:SetDefaultPolicyVersion"
      ],
      "Resource": [
        "arn:aws:iam::<ACCOUNT_ID>:policy/userboundary",
        "arn:aws:iam::<ACCOUNT_ID>:policy/adminboundary"
      ]
    },
    {
      "Sid": "NoBoundaryRemoval",
      "Effect": "Deny",
      "Action": "iam:DeleteUserPermissionsBoundary",
      "Resource": "*"
    }
  ]
}
```

## One-line Policy Summary

> Allows IAM and CloudWatch administration while making permission boundaries immutable and preventing self-escalation.

---

# 2. Allow S3ResourceManager to Assume Role

**Purpose**
Allow the IAM user **S3ResourceManager** in **Account B** to assume a specific IAM role in **Account A** using AWS STS.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Resource": "arn:aws:iam::<ACCOUNT_A_ID>:role/auditfindata"
    }
  ]
}
```

## Policy Behavior

* Grants permission to call `sts:AssumeRole`
* Scoped to **one specific role only**
* Does not grant any AWS service permissions by itself

---

## Security Insight

> This policy controls **who is allowed to attempt role assumption**, not what the role can do.

---
