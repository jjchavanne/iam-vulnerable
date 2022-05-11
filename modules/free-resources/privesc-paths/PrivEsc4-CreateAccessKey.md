#

More likely scope that a user might have.  Policy name I gave it: `IAMUserAccessKeys`

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "iam:DeleteAccessKey",
                "iam:UpdateAccessKey",
                "iam:CreateAccessKey",
                "iam:ListAccessKeys"
            ],
            "Resource": "arn:aws:iam::<ACCOUNT_ID>:user/*"
        }
    ]
}
```

Why is this is more plausible?    
- To give a user the ability to rotate access keys.  At the moment, this is set for a user, however potentially more likely cases:
    - Automation server
    - Team Admin
    - Platform Admin
    - Security Admin
