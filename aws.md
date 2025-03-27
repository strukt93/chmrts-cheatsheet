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
