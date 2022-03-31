# Getting Started


## Install minio
```bash
brew install minio/stable/minio
mkdir -p ~/minio/data
minio server ~/minio/data --console-address ":64484"
```

Got to http://127.0.0.1:64484/ and log in with minioadmin (username & password). 
Create a bucket (e.g. metaflow-test).
Then go to Identity > Users and create a new user with `Access Key` and `Secret Key`. Note these down.

Configure an AWS profile for your minio server by calling `aws configure --profile minio` and enter the `Access Key` and `Secret Key` you just set.
Enable AWS Signature version 4 for minio server:
```
aws --profile minio configure set s3.signature_version s3v4
# List buckets
aws --profile minio --endpoint-url http://127.0.0.1:9000/ s3 ls
```


## Install minikube
```bash
brew install minikube
minikube start
```

## Minio certificate
Download [certgen](https://github.com/minio/certgen).
Create certificate:
```
certgen -host "127.0.0.1,localhost,10.90.125.74"
mv public.crt ~/.minio/certs        
mv private.key ~/.minio/certs
```
Restart minio.


## Encode AWS access keys as base64 and add them to a secret.yml file
```bash
echo $AWS_ACCESS_KEY_ID | base64
echo $AWS_SECRET_ACCESS_KEY | base64
```

secret.yml
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: miniosecret
type: Opaque
data:
	METAFLOW_S3_ENDPOINT_URL: <base64 encoded>
	METAFLOW_S3_VERIFY_CERTIFICATE: <base64 encoded>
	AWS_DEFAULT_REGION: <base64 encoded>
	AWS_ACCESS_KEY_ID: <base64 encoded>
	AWS_SECRET_ACCESS_KEY: <base64 encoded>
```

Then run:
```bash
kubectl apply -f secret.yml 
# Check for success
kubectl get secret miniosecret -o yaml
```


## Launch Flow
```
AWS_PROFILE=minio python flow.py run
```