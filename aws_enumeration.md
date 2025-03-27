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
- Get a specific user/group/role's inline policy document
```
aws iam get-user-policy --user-name USERNAME --policy-name POLICYNAME
aws iam get-group-policy --group-name GROUPNAME --policy-name POLICYNAME
aws iam get-role-policy --role-name ROLENAME --policy-name POLICYNAME
```
### Enumeration for initial foothold
- Use cloud_enum to enumerate the organization
```
./cloud_enum.py -k ORGNAME
```
