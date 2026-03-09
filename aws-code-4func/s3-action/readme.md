# Amazon S3 — Basic Reference

1. What is S3
Object storage service for storing and retrieving any amount of data.
Stores data as objects in buckets. Each object has data, metadata, and a key (name).
Highly durable (11 9s), scalable, and accessible via HTTP(S) API, AWS SDKs, CLI, and console.

2. Key concepts
- Bucket: top-level container for objects; bucket names are globally unique.
- Object: file with a key (path-like name) and value (data). Max object size: 5 TB.
- Key: unique identifier for an object within a bucket (can include slashes).
- Region: physical AWS region where bucket resides — choose for latency/compliance costs.
- Prefix: common key name start used like folder names for listing.
- Versioning: keeps multiple versions of objects to protect from accidental overwrites/deletes.
- Lifecycle rules: automate transitions (e.g., Standard → Glacier) and expirations.
- Storage classes:
    - Standard — general purpose.
    - Intelligent-Tiering — auto-tiering.
    - Standard-IA — infrequent access, lower cost.
    - One Zone-IA — cheaper, single AZ.
    - Glacier Instant / Flexible / Deep Archive — archival, varying retrieval times and costs.
    - Reduced Redundancy Storage (deprecated / rarely used).
- ACLs & Policies:
    - ACLs: legacy access control at object/bucket level.
    - Bucket policies: JSON policies to allow/deny actions for principals (preferred).
    - IAM policies: attach to users/roles to control S3 access.
- Pre-signed URLs: time-limited URLs to allow temporary access to private objects.
- Server-side encryption (SSE):
    - SSE-S3 (S3-managed keys)
    - SSE-KMS (AWS KMS-managed keys)
    - SSE-C (customer-provided keys)
    - Client-side encryption: encrypt before upload.
- S3 Event notifications: send events to SQS, SNS, Lambda on object-level operations.
- Transfer acceleration: faster transfers using Amazon CloudFront edge locations.
- Multipart upload: upload large objects in parts (recommended for >100 MB; required >5 GB).
- S3 Object Lock: WORM (write-once-read-many) for compliance (retention/ legal hold).

3. Common CLI commands (AWS CLI v2)
- Configure CLI:
    - aws configure
- Create bucket:
    - aws s3api create-bucket --bucket v-bucket-name --region us-east-1
    - For non us-east-1 add --create-bucket-configuration LocationConstraint=us-west-2
- List buckets:
    - aws s3 ls
- List objects (simple):
    - aws s3 ls s3://v-bucket --recursive
- Upload a file:
    - aws s3 cp localfile.txt s3://v-bucket/path/localfile.txt
- Download a file:
    - aws s3 cp s3://v-bucket/path/localfile.txt ./localfile.txt
- Sync directory:
    - aws s3 sync ./localdir s3://v-bucket/path --delete
- Remove object:
    - aws s3 rm s3://v-bucket/path/localfile.txt
- Enable versioning:
    - aws s3api put-bucket-versioning --bucket v-bucket --versioning-configuration Status=Enabled
- Generate pre-signed URL (Python boto3 example; see SDKs too):
    - aws s3 presign s3://v-bucket/path/localfile.txt --expires-in 3600
- Set bucket policy (from file):
    - aws s3api put-bucket-policy --bucket v-bucket --policy file://policy.json

4. Basic bucket policy example (allow public read to a specific prefix)

- policy.json (concept):
```bash
{
    "Version":"2012-10-17",
    "Statement":[{
    "Effect":"Allow",
    "Principal":"",
    "Action":["s3:GetObject"],
    "Resource":["arn:aws:s3:::v-bucket/public/"]
    }]
}
```
Note: Avoid public buckets unless intentionally serving public content (e.g., static website).

5. Security best practices
- Keep buckets private by default.
- Use IAM roles for AWS services (avoid long-lived access keys).
- Prefer bucket policies and IAM policies over ACLs.
- Use SSE-KMS for sensitive data; enforce via bucket policy.
- Enable MFA Delete for extra protection (requires versioning).
- Enable S3 Block Public Access at account and bucket level.
- Monitor with AWS CloudTrail, S3 server access logs, and AWS Config.
6. Cost considerations
- Charges for:
    - Storage (per GB/month by storage class)
    - PUT/GET and other API requests
    - Data transfer out (egress)
    - Lifecycle transitions and retrievals (Glacier)
- Multipart and small object request overhead
Use lifecycle rules, intelligent-tiering, and consider object size and access patterns to optimize costs.
7. Performance & scaling tips
- Use multipart uploads for large objects.
- Distribute object keys to avoid request hot spots (S3 now handles this automatically, but avoid many requests to the same prefix).
- Use Transfer Acceleration or S3 Multipart + parallel uploads for high-throughput uploads.
- Use CloudFront in front of S3 for global low-latency read access and caching.
8. Common workflows
- Static website hosting:
    - Enable static website hosting in bucket properties.
    - Set index and error documents.
    - Optionally use custom domain with Route 53 and CloudFront (HTTPS needs CloudFront or S3 + CloudFront for certificate).
- Backup & archival:
    - Use lifecycle policies to transition to Glacier Deep Archive after X days.
- Data lake / analytics:
    - Store partitioned data with prefixes; integrate with Athena, EMR, Glue.
- Application file storage:
    - Store user uploads as objects; use pre-signed URLs for direct browser upload/download.

9. Useful links & docs
- S3 Developer Guide: https://docs.aws.amazon.com/AmazonS3/latest/dev/Welcome.html
- S3 Pricing: https://aws.amazon.com/s3/pricing/
AWS CLI S3: https://awscli.amazonaws.com/v2/documentation/api/latest/reference/s3/index.html
- AWS SDKs: https://aws.amazon.com/tools/

10. Quick examples (curl & presigned URL)
- Upload with presigned PUT URL (from server generates URL then client uses):
    - Client: curl -T file.txt "https://presigned-url" -H "Content-Type: text/plain"
- Download via presigned GET:
    - curl "https://presigned-url" -o file.txt

## Configuration

> `args` would be passed directly to `aws s3` invocation. Refer to: [AWS S3 docs](https://docs.aws.amazon.com/cli/latest/userguide/cli-services-s3.html)

| Environment Variable       | Required                    | Description                                                                                                      | 
|----------------------------|-----------------------------|------------------------------------------------------------------------------------------------------------------|
| AWS_ACCESS_KEY_ID          | true                        | Access Key ID. A unique identifier that's associated with a secret access key                                    |
| AWS_SECRET_ACCESS_KEY      | true                        | Secret Access Key. A key that's used with the access key ID to cryptographically sign programmatic AWS requests. |
| AWS_REGION                 | true                        | Your AWS Region                                                                                                  |
| AWS_S3_BUCKET              | true                        | Your S3 Bucket name                                                                                              |
| S3_SOURCE_DIR              | true                        | Path to local folder                                                                                             |
| DESTINATION_DIR            | false (defaults to S3 root) | Path to folder in a S3 bucket                                                                                    |
| CLOUDFRONT_DISTRIBUTION_ID | false                       | Your CloudFront distribution                                                                                     |
| CLOUDFRONT_PATHS           | false (defaults to /*)      | The paths to invalidate (relative to the distribution)                                                           |
| USE_GZIP_COMPRESSION       | false (defaults to false)   | Use gzip for upload                                                                                              |