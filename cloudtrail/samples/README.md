# samples
CloudTrail samples

# Compress and store in local bucket

Compress the JSON file and store it in the local S3 bucket:
```bash
gzip -c aws-ec2-start-instances.json > 9876_CloudTrail_us-east-1_20240501T0000Z_abc.json.gz
```

Copy the compressed file to the S3 bucket:
```bash
aws --endpoint-url http://127.0.0.1:9000 \
  s3 cp 9876_CloudTrail_us-east-1_20240501T0000Z_abc.json.gz \
  s3://my-local-bucket-falcolab/AWSLogs/o-1234/9876/CloudTrail/us-east-1/2024/05/01/
```

# References
- [CloudTrail Log File Examples](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-log-file-examples.html)
