# S3 Storage Configuration Guide for SciNote

This guide provides detailed instructions for configuring AWS S3 and S3-compatible storage services with SciNote.

## 📦 AWS S3 Configuration

### Step 1: Create S3 Bucket

1. **Log in to AWS Console**
   - Navigate to S3 service
   - Click "Create bucket"

2. **Bucket Settings**
   ```
   Bucket name: scinote-production-storage
   AWS Region: us-east-1 (or your preferred region)
   
   Object Ownership: ACLs disabled (recommended)
   
   Block Public Access settings:
   ✅ Block all public access (recommended)
   
   Bucket Versioning: Enable (recommended)
   
   Default encryption:
   ✅ Enable
   Encryption type: SSE-S3 or SSE-KMS
   
   Object Lock: Disabled (unless required)
   ```

3. **Click "Create bucket"**

### Step 2: Configure Bucket Policy (Optional)

For additional security, add a bucket policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyInsecureTransport",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::scinote-production-storage",
        "arn:aws:s3:::scinote-production-storage/*"
      ],
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      }
    },
    {
      "Sid": "AllowSciNoteAccess",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::YOUR_ACCOUNT_ID:user/scinote-s3-user"
      },
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject",
        "s3:ListBucket",
        "s3:GetObjectVersion"
      ],
      "Resource": [
        "arn:aws:s3:::scinote-production-storage",
        "arn:aws:s3:::scinote-production-storage/*"
      ]
    }
  ]
}
```

### Step 3: Configure CORS (If Needed)

If you need direct browser uploads to S3:

1. Go to bucket → Permissions → CORS
2. Add CORS configuration:

```json
[
  {
    "AllowedHeaders": [
      "*"
    ],
    "AllowedMethods": [
      "GET",
      "PUT",
      "POST",
      "DELETE",
      "HEAD"
    ],
    "AllowedOrigins": [
      "https://your-scinote-domain.com"
    ],
    "ExposeHeaders": [
      "ETag",
      "x-amz-request-id"
    ],
    "MaxAgeSeconds": 3000
  }
]
```

### Step 4: Create IAM User

1. **Navigate to IAM → Users**
   - Click "Add users"
   - User name: `scinote-s3-user`
   - Access type: ✅ Programmatic access
   - Click "Next"

2. **Set Permissions**
   - Select "Attach policies directly"
   - Click "Create policy"
   - Use the JSON editor:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "SciNoteS3Access",
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:PutObjectAcl",
        "s3:GetObject",
        "s3:GetObjectAcl",
        "s3:DeleteObject",
        "s3:ListBucket",
        "s3:GetBucketLocation",
        "s3:GetObjectVersion",
        "s3:ListBucketVersions"
      ],
      "Resource": [
        "arn:aws:s3:::scinote-production-storage",
        "arn:aws:s3:::scinote-production-storage/*"
      ]
    }
  ]
}
```

3. **Name the Policy**
   - Policy name: `SciNoteS3Policy`
   - Click "Create policy"

4. **Attach Policy to User**
   - Go back to user creation
   - Refresh policies and select `SciNoteS3Policy`
   - Click "Next" → "Create user"

5. **Save Credentials**
   - **Access Key ID**: Copy and save securely
   - **Secret Access Key**: Copy and save securely
   - ⚠️ You won't be able to see the secret key again!

### Step 5: Configure SciNote Environment

Add these variables to your `.env` file or Easypanel:

```env
# AWS S3 Configuration
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
AWS_REGION=us-east-1
S3_BUCKET=scinote-production-storage
S3_SUBFOLDER=scinote
ACTIVESTORAGE_SERVICE=amazon
```

## 🌐 S3-Compatible Services Overview

This deployment supports all S3-compatible storage providers. The configuration uses AWS SDK with custom endpoints to connect to various providers.

### Supported Providers

The following S3-compatible services are fully supported:

| Provider | Endpoint Pattern | Path Style Required | Notes |
|----------|------------------|---------------------|-------|
| AWS S3 | Default AWS | No | Native support |
| DigitalOcean Spaces | `https://{region}.digitaloceanspaces.com` | Yes | Multiple regions |
| Wasabi | `https://s3.wasabisys.com` | Yes | Cost-effective |
| MinIO | `http://your-server:9000` | Yes | Self-hosted |
| Backblaze B2 | `https://s3.{region}.backblazeb2.com` | Yes | S3-compatible API |
| Google Cloud Storage | `https://storage.googleapis.com` | Yes | Via interoperability |
| Oracle Cloud OSS | `https://objectstorage.{region}.oraclecloud.com` | Yes | Enterprise-grade |
| Scaleway Object Storage | `https://s3.{region}.scw.cloud` | Yes | European provider |
| Linode Object Storage | `https://{region}.linodeobjects.com` | Yes | Akamai provider |
| Alibaba Cloud OSS | `https://oss-{region}.aliyuncs.com` | Yes | Asian regions |
| IDrive e2 | `https://e2.wasabisys.com` | Yes | Wasabi-powered |

### Configuration Requirements

For all S3-compatible services, you need to set these environment variables:

```env
ACTIVESTORAGE_SERVICE=amazon
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
AWS_REGION=your_region
AWS_ENDPOINT=provider_specific_endpoint
AWS_FORCE_PATH_STYLE=true  # Required for most non-AWS providers
S3_BUCKET=your_bucket_name
S3_SUBFOLDER=scinote  # Optional
```

### DigitalOcean Spaces

1. **Create Space**
   - Go to DigitalOcean → Spaces
   - Click "Create Space"
   - Choose datacenter region
   - Space name: `scinote-storage`

2. **Generate API Keys**
   - Go to API → Spaces Keys
   - Click "Generate New Key"
   - Save Access Key and Secret Key

3. **Configure SciNote**

```env
AWS_ACCESS_KEY_ID=your_spaces_access_key
AWS_SECRET_ACCESS_KEY=your_spaces_secret_key
AWS_REGION=us-east-1
AWS_ENDPOINT=https://nyc3.digitaloceanspaces.com
S3_BUCKET=scinote-storage
S3_SUBFOLDER=scinote
ACTIVESTORAGE_SERVICE=amazon
```

**Note**: Replace `nyc3` with your actual region (e.g., `sfo3`, `ams3`, `sgp1`)

### Wasabi

1. **Create Bucket**
   - Log in to Wasabi Console
   - Create new bucket
   - Choose region

2. **Create Access Keys**
   - Go to Access Keys
   - Create new access key
   - Save credentials

3. **Configure SciNote**

```env
AWS_ACCESS_KEY_ID=your_wasabi_access_key
AWS_SECRET_ACCESS_KEY=your_wasabi_secret_key
AWS_REGION=us-east-1
AWS_ENDPOINT=https://s3.us-east-1.wasabisys.com
S3_BUCKET=scinote-storage
S3_SUBFOLDER=scinote
ACTIVESTORAGE_SERVICE=amazon
```

**Wasabi Endpoints by Region:**
- US East 1: `https://s3.us-east-1.wasabisys.com`
- US East 2: `https://s3.us-east-2.wasabisys.com`
- US West 1: `https://s3.us-west-1.wasabisys.com`
- EU Central 1: `https://s3.eu-central-1.wasabisys.com`

### Backblaze B2

1. **Create Bucket**
   - Log in to Backblaze
   - Create new B2 bucket
   - Set to private

2. **Create Application Key**
   - Go to App Keys
   - Create new key with access to your bucket
   - Save Key ID and Application Key

3. **Configure SciNote**

```env
AWS_ACCESS_KEY_ID=your_b2_key_id
AWS_SECRET_ACCESS_KEY=your_b2_application_key
AWS_REGION=us-west-000
AWS_ENDPOINT=https://s3.us-west-000.backblazeb2.com
S3_BUCKET=scinote-storage
S3_SUBFOLDER=scinote
ACTIVESTORAGE_SERVICE=amazon
```

**Note**: Replace `us-west-000` with your actual endpoint from Backblaze

### Google Cloud Storage

1. **Enable S3 Interoperability**
   - Go to Google Cloud Console
   - Navigate to Cloud Storage
   - Settings → Interoperability
   - Enable "Interoperable Access"
   - Create HMAC keys
   - Save Access Key and Secret

2. **Create Bucket**
   - Create new bucket in desired region
   - Set appropriate permissions

3. **Configure SciNote**

```env
AWS_ACCESS_KEY_ID=your_gcs_access_key
AWS_SECRET_ACCESS_KEY=your_gcs_secret_key
AWS_REGION=auto
AWS_ENDPOINT=https://storage.googleapis.com
AWS_FORCE_PATH_STYLE=true
S3_BUCKET=scinote-storage
S3_SUBFOLDER=scinote
ACTIVESTORAGE_SERVICE=amazon
```

### Oracle Cloud Object Storage

1. **Create Bucket**
   - Log in to Oracle Cloud Console
   - Navigate to Object Storage
   - Create bucket in desired compartment

2. **Create Customer Secret Key**
   - Go to User Settings → Customer Secret Keys
   - Create new key
   - Save Access Key and Secret

3. **Configure SciNote**

```env
AWS_ACCESS_KEY_ID=your_oracle_access_key
AWS_SECRET_ACCESS_KEY=your_oracle_secret_key
AWS_REGION=us-ashburn-1
AWS_ENDPOINT=https://objectstorage.us-ashburn-1.oraclecloud.com
AWS_FORCE_PATH_STYLE=true
S3_BUCKET=scinote-storage
S3_SUBFOLDER=scinote
ACTIVESTORAGE_SERVICE=amazon
```

### Scaleway Object Storage

1. **Create Bucket**
   - Log in to Scaleway Console
   - Navigate to Object Storage
   - Create bucket in desired region

2. **Create S3 Credentials**
   - Go to API Keys
   - Create S3 credentials
   - Save Access Key and Secret

3. **Configure SciNote**

```env
AWS_ACCESS_KEY_ID=your_scaleway_access_key
AWS_SECRET_ACCESS_KEY=your_scaleway_secret_key
AWS_REGION=fr-par
AWS_ENDPOINT=https://s3.fr-par.scw.cloud
AWS_FORCE_PATH_STYLE=true
S3_BUCKET=scinote-storage
S3_SUBFOLDER=scinote
ACTIVESTORAGE_SERVICE=amazon
```

### Linode Object Storage

1. **Create Bucket**
   - Log in to Linode Cloud Manager
   - Navigate to Object Storage
   - Create bucket in desired region

2. **Create Access Key**
   - Go to Access Keys
   - Create new key
   - Save Access Key and Secret

3. **Configure SciNote**

```env
AWS_ACCESS_KEY_ID=your_linode_access_key
AWS_SECRET_ACCESS_KEY=your_linode_secret_key
AWS_REGION=us-southeast-1
AWS_ENDPOINT=https://us-southeast-1.linodeobjects.com
AWS_FORCE_PATH_STYLE=true
S3_BUCKET=scinote-storage
S3_SUBFOLDER=scinote
ACTIVESTORAGE_SERVICE=amazon
```

### MinIO (Self-Hosted)

1. **Install MinIO**

```bash
# Using Docker
docker run -d \
  -p 9000:9000 \
  -p 9001:9001 \
  --name minio \
  -v /data/minio:/data \
  -e "MINIO_ROOT_USER=minioadmin" \
  -e "MINIO_ROOT_PASSWORD=minioadmin" \
  minio/minio server /data --console-address ":9001"
```

## 🔐 Security Best Practices

### 1. Bucket Access Control

- **Keep buckets private**: Never make buckets publicly accessible
- **Use IAM policies**: Grant minimal required permissions
- **Enable versioning**: Protect against accidental deletions
- **Enable logging**: Track access to your bucket

### 2. Encryption

**Server-Side Encryption (SSE-S3)**
```
Default encryption: Enabled
Encryption type: SSE-S3
```

**Server-Side Encryption with KMS (SSE-KMS)**
```
Default encryption: Enabled
Encryption type: SSE-KMS
KMS key: Choose or create KMS key
```

### 3. Access Key Management

- **Rotate keys regularly**: Change access keys every 90 days
- **Use separate keys**: Different keys for dev/staging/production
- **Never commit keys**: Keep credentials out of version control
- **Use IAM roles**: When possible, use IAM roles instead of keys

### 4. Bucket Policies

**Deny unencrypted uploads:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyUnencryptedObjectUploads",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::scinote-production-storage/*",
      "Condition": {
        "StringNotEquals": {
          "s3:x-amz-server-side-encryption": "AES256"
        }
      }
    }
  ]
}
```

### 5. Lifecycle Policies

Configure lifecycle rules to manage costs:

```json
{
  "Rules": [
    {
      "Id": "MoveToIA",
      "Status": "Enabled",
      "Transitions": [
        {
          "Days": 90,
          "StorageClass": "STANDARD_IA"
        }
      ],
      "NoncurrentVersionTransitions": [
        {
          "NoncurrentDays": 30,
          "StorageClass": "STANDARD_IA"
        }
      ]
    }
  ]
}
```

## 📊 Monitoring and Costs

### AWS CloudWatch Metrics

Monitor these S3 metrics:
- **BucketSizeBytes**: Total bucket size
- **NumberOfObjects**: Object count
- **AllRequests**: Total requests
- **GetRequests**: Download requests
- **PutRequests**: Upload requests

### Cost Optimization

1. **Use Lifecycle Policies**: Move old files to cheaper storage classes
2. **Enable Intelligent-Tiering**: Automatic cost optimization
3. **Delete Unused Files**: Regular cleanup of temporary files
4. **Monitor Transfer Costs**: Track data transfer out of S3
5. **Use CloudFront**: CDN for frequently accessed files

### Cost Estimation

**Example Monthly Costs (AWS S3 us-east-1):**
```
Storage: 100 GB × $0.023/GB = $2.30
PUT requests: 10,000 × $0.005/1000 = $0.05
GET requests: 100,000 × $0.0004/1000 = $0.04
Data transfer out: 50 GB × $0.09/GB = $4.50
Total: ~$7/month
```

## 🧪 Testing S3 Configuration

### Test Upload via Rails Console

```ruby
# Access Rails console
docker-compose exec web rails console

# Test S3 upload
blob = ActiveStorage::Blob.create_and_upload!(
  io: StringIO.new("Test content"),
  filename: "test.txt",
  content_type: "text/plain"
)

# Verify upload
puts blob.service_url

# Clean up
blob.purge
```

### Test via AWS CLI

```bash
# Configure AWS CLI
aws configure

# List buckets
aws s3 ls

# Upload test file
echo "test" > test.txt
aws s3 cp test.txt s3://scinote-production-storage/test/

# Download test file
aws s3 cp s3://scinote-production-storage/test/test.txt downloaded.txt

# Delete test file
aws s3 rm s3://scinote-production-storage/test/test.txt
```

## 🔧 Troubleshooting

### Issue: Access Denied

**Symptoms:**
- 403 Forbidden errors
- "Access Denied" in logs

**Solutions:**
1. Verify IAM permissions include all required actions
2. Check bucket policy doesn't deny access
3. Verify access keys are correct
4. Ensure bucket exists and name is correct

### Issue: Connection Timeout

**Symptoms:**
- Timeout errors when uploading
- Slow upload speeds

**Solutions:**
1. Check network connectivity
2. Verify endpoint URL is correct
3. Check firewall rules
4. Try different AWS region

### Issue: Invalid Signature

**Symptoms:**
- "SignatureDoesNotMatch" error
- Authentication failures

**Solutions:**
1. Verify secret access key is correct
2. Check for extra spaces in credentials
3. Ensure system time is synchronized
4. Regenerate access keys if needed

### Issue: Bucket Not Found

**Symptoms:**
- "NoSuchBucket" error
- 404 errors

**Solutions:**
1. Verify bucket name is correct
2. Check bucket exists in specified region
3. Ensure region matches `AWS_REGION` variable
4. Check bucket hasn't been deleted

## 📚 Additional Resources

- [AWS S3 Documentation](https://docs.aws.amazon.com/s3/)
- [AWS S3 Pricing](https://aws.amazon.com/s3/pricing/)
- [Rails Active Storage Guide](https://guides.rubyonrails.org/active_storage_overview.html)
- [DigitalOcean Spaces Docs](https://docs.digitalocean.com/products/spaces/)
- [Wasabi Documentation](https://wasabi-support.zendesk.com/)
- [MinIO Documentation](https://min.io/docs/minio/linux/index.html)

## ✅ Configuration Checklist

- [ ] S3 bucket created
- [ ] Bucket encryption enabled
- [ ] Bucket versioning enabled (optional)
- [ ] IAM user created with proper permissions
- [ ] Access keys generated and saved securely
- [ ] Bucket policy configured (if needed)
- [ ] CORS configured (if needed)
- [ ] Environment variables set in SciNote
- [ ] Upload test completed successfully
- [ ] Monitoring configured
- [ ] Backup strategy defined
- [ ] Cost alerts set up

---

Your S3 storage is now configured and ready for use with SciNote! 🎉