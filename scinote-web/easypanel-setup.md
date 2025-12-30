# Easypanel Deployment Guide for SciNote

This guide provides step-by-step instructions for deploying SciNote on Easypanel with S3 storage.

## 📋 Pre-Deployment Checklist

- [ ] Easypanel instance is running and accessible
- [ ] AWS account with S3 bucket created
- [ ] IAM user with S3 permissions created
- [ ] Domain name configured (optional but recommended)
- [ ] SSL certificate ready (Easypanel can handle this automatically)

## 🚀 Deployment Steps

### Step 1: Prepare AWS S3

1. **Create S3 Bucket**
   - Go to AWS Console → S3
   - Click "Create bucket"
   - Bucket name: `scinote-production-storage` (or your preferred name)
   - Region: Choose closest to your users
   - Block all public access: ✅ (recommended)
   - Bucket versioning: Enable (recommended)
   - Default encryption: Enable

2. **Create IAM User**
   - Go to AWS Console → IAM → Users
   - Click "Add users"
   - User name: `scinote-s3-user`
   - Access type: Programmatic access
   - Attach policy directly → Create policy:

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
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

3. **Save Credentials**
   - Copy Access Key ID
   - Copy Secret Access Key
   - Store securely (you'll need these for Easypanel)

### Step 2: Configure Easypanel Project

1. **Create New Project**
   - Log in to Easypanel
   - Click "Create Project"
   - Project name: `scinote`
   - Click "Create"

2. **Add Docker Compose Service**
   - In your project, click "Add Service"
   - Select "Docker Compose"
   - Service name: `scinote-app`

3. **Paste Docker Compose Configuration**
   - Copy the entire contents of `docker-compose.yml`
   - Paste into the Easypanel editor
   - Click "Save"

### Step 3: Configure Environment Variables

In Easypanel, add the following environment variables:

#### Required Variables

```
SECRET_KEY_BASE=<generate with: openssl rand -hex 64>
POSTGRES_PASSWORD=<strong_password_here>
AWS_ACCESS_KEY_ID=<your_aws_access_key>
AWS_SECRET_ACCESS_KEY=<your_aws_secret_key>
AWS_REGION=us-east-1
S3_BUCKET=scinote-production-storage
APPLICATION_URL=https://scinote.yourdomain.com
WEB_PORT=37842
POSTGRES_PORT=54329
REDIS_PORT=63791
```

#### Optional Variables

```
S3_SUBFOLDER=scinote
APPLICATION_NAME=SciNote Lab
MAIL_SERVER_URL=smtp.gmail.com
MAIL_SERVER_PORT=587
MAIL_SERVER_USERNAME=your_email@gmail.com
MAIL_SERVER_PASSWORD=your_app_password
MAIL_FROM=noreply@yourdomain.com
WEB_PORT=37842
DB_POOL=25
RAILS_MAX_THREADS=5
WEB_CONCURRENCY=2
```

### Step 4: Configure Domain and SSL

1. **Add Domain**
   - In Easypanel, go to your service settings
   - Click "Domains"
   - Add your domain: `scinote.yourdomain.com`
   - Easypanel will automatically provision SSL certificate

2. **DNS Configuration**
   - Add A record pointing to your Easypanel server IP
   - Or add CNAME record if using subdomain

### Step 5: Deploy the Application

1. **Start Deployment**
   - Click "Deploy" in Easypanel
   - Wait for all services to start (this may take 5-10 minutes)
   - Monitor logs for any errors

2. **Initialize Database**
   - Once deployed, open the web service terminal in Easypanel
   - Run the following commands:

   ```bash
   rake db:create
   rake db:migrate
   rake db:seed
   ```

   Or use the Easypanel console:
   - Select the `web` service
   - Click "Console"
   - Run the commands above

### Step 6: Verify Deployment

1. **Check Service Health**
   - In Easypanel, verify all services are running:
     - ✅ db (PostgreSQL)
     - ✅ redis
     - ✅ web
     - ✅ jobs

2. **Test Application**
   - Open your domain in a browser
   - You should see the SciNote login page
   - Create an admin account

3. **Test S3 Upload**
   - Log in to SciNote
   - Try uploading an image or file
   - Verify it appears in your S3 bucket

## 🔧 Post-Deployment Configuration

### Create Admin User

If you need to create an admin user via console:

```bash
# Access the web service console in Easypanel
rails console

# Create admin user
User.create!(
  full_name: 'Admin User',
  email: 'admin@yourdomain.com',
  password: 'secure_password',
  password_confirmation: 'secure_password',
  confirmed_at: Time.now
)
```

### Configure Email Settings

For production use, configure SMTP settings in environment variables:

```
MAIL_SERVER_URL=smtp.sendgrid.net
MAIL_SERVER_PORT=587
MAIL_SERVER_USERNAME=apikey
MAIL_SERVER_PASSWORD=<your_sendgrid_api_key>
MAIL_FROM=noreply@yourdomain.com
```

### Enable Additional Features

Add these environment variables to enable features:

```
ACTIVESTORAGE_ENABLE_PDF_PREVIEWS=true
ACTIVESTORAGE_ENABLE_VIPS=true
```

## 📊 Monitoring and Maintenance

### View Logs in Easypanel

1. Go to your service
2. Click "Logs"
3. Select the service you want to monitor:
   - `web` - Application logs
   - `jobs` - Background job logs
   - `db` - Database logs
   - `redis` - Redis logs

### Database Backups

Set up automated backups in Easypanel:

1. Go to service settings
2. Click "Backups"
3. Configure backup schedule
4. Set retention policy

Or manually backup:

```bash
# In Easypanel console for db service
pg_dump -U postgres scinote_production > /backup/scinote_$(date +%Y%m%d).sql
```

### Update SciNote

To update to the latest version:

1. In Easypanel, go to your service
2. Click "Rebuild"
3. This will pull the latest image
4. After rebuild, run migrations:
   ```bash
   rake db:migrate
   ```

## 🔒 Security Hardening

### 1. Restrict Database Access

In `docker-compose.yml`, database and Redis ports are exposed for management:
- PostgreSQL on port 54329 (mapped from internal 5432)
- Redis on port 63791 (mapped from internal 6379)
- For production, consider removing port mappings if external access is not needed
- Use firewall rules to restrict access to these ports

### 2. Use Strong Passwords

- Generate strong `SECRET_KEY_BASE`: `openssl rand -hex 64`
- Use complex database password
- Rotate credentials regularly

### 3. Configure S3 Bucket Policy

Add bucket policy to restrict access:

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
    }
  ]
}
```

### 4. Enable S3 Encryption

- Go to S3 bucket properties
- Enable default encryption
- Choose SSE-S3 or SSE-KMS

### 5. Configure CORS (if needed)

If accessing S3 directly from browser:

```json
[
  {
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["GET", "PUT", "POST", "DELETE"],
    "AllowedOrigins": ["https://scinote.yourdomain.com"],
    "ExposeHeaders": ["ETag"]
  }
]
```

## 🐛 Troubleshooting

### Issue: Services Won't Start

**Solution:**
1. Check Easypanel logs for errors
2. Verify all environment variables are set
3. Ensure `SECRET_KEY_BASE` is generated
4. Check database connection

### Issue: S3 Upload Fails

**Solution:**
1. Verify AWS credentials in environment variables
2. Check S3 bucket exists and is accessible
3. Verify IAM permissions
4. Check bucket region matches `AWS_REGION`
5. Review web service logs for S3 errors

### Issue: Database Connection Error

**Solution:**
1. Verify database service is running
2. Check `POSTGRES_PASSWORD` matches in all services
3. Ensure database has been initialized
4. Try restarting the database service

### Issue: Cannot Access Application

**Solution:**
1. Verify domain DNS is configured correctly
2. Check SSL certificate is provisioned
3. Ensure port 3000 is exposed in docker-compose
4. Check Easypanel proxy configuration

## 📈 Scaling

### Horizontal Scaling

To scale web workers in Easypanel:

1. Edit `docker-compose.yml`
2. Add deploy configuration:

```yaml
web:
  # ... existing configuration
  deploy:
    replicas: 3
```

### Vertical Scaling

Increase resources in Easypanel:
1. Go to service settings
2. Adjust CPU and memory limits
3. Restart service

### Database Scaling

For larger deployments:
1. Use managed PostgreSQL (AWS RDS, DigitalOcean Managed DB)
2. Update `DATABASE_URL` to point to managed instance
3. Remove `db` service from docker-compose

## 🎯 Best Practices

1. **Regular Backups**: Set up automated database and S3 backups
2. **Monitoring**: Use Easypanel monitoring or external tools
3. **Updates**: Keep SciNote updated to latest stable version
4. **Security**: Regularly review and update credentials
5. **Testing**: Test backups and disaster recovery procedures
6. **Documentation**: Keep deployment notes and configurations documented

## 📞 Support Resources

- **Easypanel Docs**: https://easypanel.io/docs
- **SciNote Community**: https://community.scinote.net
- **AWS S3 Docs**: https://docs.aws.amazon.com/s3/
- **Docker Compose**: https://docs.docker.com/compose/

## ✅ Deployment Checklist

- [ ] S3 bucket created and configured
- [ ] IAM user created with proper permissions
- [ ] Easypanel project created
- [ ] Docker Compose configuration added
- [ ] All environment variables configured
- [ ] Domain and SSL configured
- [ ] Application deployed successfully
- [ ] Database initialized
- [ ] Admin user created
- [ ] S3 upload tested
- [ ] Email configuration tested (if applicable)
- [ ] Backups configured
- [ ] Monitoring set up
- [ ] Documentation updated

---

**Congratulations!** Your SciNote instance should now be running on Easypanel with S3 storage. 🎉