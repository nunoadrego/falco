# falco
Playing with Falco and AWS CloudTrail

# Environment

This lab runs locally. We will simulate a scenario in which CloudTrail writes to an S3 bucket and
Falco reads from that bucket.

AWS:
- Organization ID: `o-1234`
- AWS Accounts: `9876`, `4321`

# Requirements
- kind
- Helm
- kubectl
- aws cli
- gzip

# Create Kubernetes cluster

Create Kubernetes cluster:

```bash
kind create cluster --config ./kind.yaml
```

# Deploy MinIO

Install MinIO in Kubernetes cluster with Helm:

```bash
helm repo add minio https://charts.min.io/
helm upgrade minio minio/minio \
  --install \
  --namespace minio \
  --create-namespace \
  --values ./minio-values.yaml \
  --kube-context kind-falcolab
```

In a different terminal, run:

```bash
export POD_NAME=$(kubectl get pods --namespace minio -l "release=minio" -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward $POD_NAME 9000 9001 --namespace minio
```

## Create bucket

Set AWS credentials (check `minio-values.yaml`):

```bash
export AWS_DEFAULT_REGION=us-east-1
export AWS_ACCESS_KEY_ID=rootuser
export AWS_SECRET_ACCESS_KEY=rootpass123
```

Create and list bucket:

```bash
aws --endpoint-url http://127.0.0.1:9000 s3 mb s3://my-local-bucket-falcolab
aws --endpoint-url http://127.0.0.1:9000 s3 ls
```

## Populate bucket

Follow the procedure in the [samples](samples/README.md) folder.

# Deploy Falco 

Install Falco in Kubernetes cluster with Helm:

```
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm upgrade falco falcosecurity/falco \
    --install \
    --create-namespace \
    --namespace falco \
    --values ./falco-values.yaml \
    --kube-context kind-falcolab
```

Note: Falco will be in a `CrashLoopBackOff` state, since this plugins is configured to read all
objects from the bucket a single time. You can read more about this configuration [here](https://github.com/falcosecurity/plugins/blob/main/plugins/cloudtrail/README.md#read-from-s3-bucket-directly). In a production
environment, it is better to use the [read from SQS queue](https://github.com/falcosecurity/plugins/blob/main/plugins/cloudtrail/README.md#read-from-sqs-queue) mode.

# Check logs of Falco

Falco is running with a rule that alerts on all CloudTrail events. As a result, you can see that with:

```
export FALCO_POD_NAME=$(kubectl get pods --namespace falco -o jsonpath="{.items[0].metadata.name}")
kubectl logs $FALCO_POD_NAME -n falco
```

# Cleanup

```
kind delete cluster --name falcolab
```

# References

## MinIO
- [MinIO Helm Chart](https://github.com/minio/minio/tree/master/helm/minio)
- [MinIO Kubernetes Documentation](https://min.io/docs/minio/kubernetes/upstream/index.html)
- [AWS CLI with MinIO](https://min.io/docs/minio/linux/integrations/aws-cli-with-minio.html)

## Falco
- [Falco Helm Chart](https://github.com/falcosecurity/charts/tree/master/charts/falco)
- [Falco CloudTrail Plugin](https://github.com/falcosecurity/plugins/tree/main/plugins/cloudtrail)


