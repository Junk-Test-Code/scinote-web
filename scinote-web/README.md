# SciNote Docker Compose for Easypanel with S3 Storage

This Docker Compose configuration deploys SciNote ELN (Electronic Lab Notebook) on Easypanel with AWS S3 storage for uploaded images and files.

## 📋 Prerequisites

- Docker and Docker Compose installed
- Easypanel instance running
- AWS account with S3 bucket created
- AWS IAM credentials with S3 access

## 🚀 Quick Start

### 1. Clone or Copy Files

Copy the following files to your Easypanel project:
- `docker-compose.yml`
- `.env.example`

### 2. Configure Environment Variables

```bash
# Copy the example environment file
cp .env.example .env

# Edit the .env file with your actual values
nano .env
```

**Required Configuration:**

```env
# Generate a secure secret key
SECRET_KEY_BASE=$(openssl rand -hex 64)

# Database credentials
POSTGRES_PASSWORD=your_secure_password

# AWS S3 Configuration
AWS_ACCESS_KEY_ID=your_aws_access_key_id
AWS_SECRET_ACCESS_KEY=your_aws_secret_access_key
AWS_REGION=us-east-1
S3_BUCKET=your-scinote-bucket

# Application URL
APPLICATION_URL=https://your-domain.com
```

### 3. Create S3 Bucket

1. Log in to AWS Console
2. Navigate to S3
3. Create a new bucket (e.g., `my-scinote-storage`)
4. Configure bucket settings:
   - **Block Public Access**: Keep enabled (recommended)
   - **Versioning**: Enable (recommended for data protection)
   - **Encryption**: Enable server-side encryption

### 4. Configure IAM Permissions

Create an IAM user with the following policy:

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
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::your-scinote-bucket",
        "arn:aws:s3:::your-scinote-bucket/*"
      ]
    }
  ]
}
```

### 5. Deploy on Easypanel

#### Option A: Using Easypanel UI

1. Create a new project in Easypanel
2. Add a new service using "Docker Compose"
3. Paste the contents of `docker-compose.yml`
4. Add environment variables from your `.env` file
5. Deploy the service

#### Option B: Using CLI

```bash
# Deploy the stack
docker-compose up -d

# Check service status
docker-compose ps

# View logs
docker-compose logs -f web
```

### 6. Initialize Database

After the first deployment, initialize the database:

```bash
# Run database migrations
docker-compose exec web rake db:create db:migrate

# Seed initial data (optional)
docker-compose exec web rake db:seed
```

### 7. Access SciNote

Open your browser and navigate to:
- Local: `http://localhost:37842`
- Production: Your configured `APPLICATION_URL`

## 🏗️ Architecture

The deployment consists of four services:

1. **PostgreSQL Database** (`db`)
   - Stores application data
   - Persistent volume for data retention
   - Port 54329 exposed (mapped to internal port 5432)

2. **Redis** (`redis`)
   - Handles Action Cable (WebSocket) connections
   - Caching layer
   - Port 63791 exposed (mapped to internal port 6379)

3. **Web Application** (`web`)
   - Main SciNote application
   - Serves HTTP requests
   - Port 37842 exposed (mapped to internal port 3000)

4. **Background Jobs** (`jobs`)
   - Processes delayed jobs
   - Handles async tasks

## 📦 Storage Configuration

### S3 Storage (Recommended for Production)

All uploaded files (images, documents, attachments) are stored in S3-compatible storage:

- **Service**: Any S3-compatible provider (AWS, DigitalOcean, Wasabi, MinIO, etc.)
- **Configuration**: Via environment variables
- **Benefits**:
  - Scalable storage
  - High availability
  - Cost-effective
  - No local disk space concerns

### S3-Compatible Services

The deployment supports all S3-compatible services including:

#### AWS S3
```env
ACTIVESTORAGE_SERVICE=amazon
AWS_ACCESS_KEY_ID=your_aws_access_key
AWS_SECRET_ACCESS_KEY=your_aws_secret_key
AWS_REGION=us-east-1
S3_BUCKET=your-bucket-name
```

#### DigitalOcean Spaces
```env
ACTIVESTORAGE_SERVICE=amazon
AWS_ACCESS_KEY_ID=your_spaces_key
AWS_SECRET_ACCESS_KEY=your_spaces_secret
AWS_REGION=us-east-1
AWS_ENDPOINT=https://nyc3.digitaloceanspaces.com
AWS_FORCE_PATH_STYLE=true
S3_BUCKET=your-space-name
```

#### Wasabi
```env
ACTIVESTORAGE_SERVICE=amazon
AWS_ACCESS_KEY_ID=your_wasabi_key
AWS_SECRET_ACCESS_KEY=your_wasabi_secret
AWS_REGION=us-east-1
AWS_ENDPOINT=https://s3.wasabisys.com
AWS_FORCE_PATH_STYLE=true
S3_BUCKET=your-bucket-name
```

#### MinIO (Self-hosted)
```env
ACTIVESTORAGE_SERVICE=amazon
AWS_ACCESS_KEY_ID=your_minio_key
AWS_SECRET_ACCESS_KEY=your_minio_secret
AWS_REGION=us-east-1
AWS_ENDPOINT=http://minio-server:9000
AWS_FORCE_PATH_STYLE=true
S3_BUCKET=your-bucket-name
```

#### Other S3-Compatible Services
Supports: Backblaze B2, Google Cloud Storage, Oracle Cloud, Scaleway, Linode, IDrive e2, and more. See `.env.example` for complete configuration examples.

### Local Storage Fallback

If S3 is not configured, files will be stored locally in Docker volumes:
- `storage_data`: Main storage volume
- `public_system`: Public file system

**Note**: Local storage is not recommended for production as it doesn't scale across multiple instances.

## 🔧 Configuration Options

### Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `SECRET_KEY_BASE` | Yes | - | Rails secret key for encryption |
| `POSTGRES_PASSWORD` | Yes | - | Database password |
| `ACTIVESTORAGE_SERVICE` | Yes* | amazon | Storage service (amazon, local) |
| `AWS_ACCESS_KEY_ID` | Yes* | - | S3 access key |
| `AWS_SECRET_ACCESS_KEY` | Yes* | - | S3 secret key |
| `S3_BUCKET` | Yes* | - | S3 bucket name |
| `AWS_REGION` | No | us-east-1 | S3 region |
| `S3_REGION` | No | us-east-1 | S3 specific region |
| `AWS_ENDPOINT` | No | - | Custom S3 endpoint for S3-compatible services |
| `AWS_FORCE_PATH_STYLE` | No | false | Force path style for S3-compatible services |
| `S3_SUBFOLDER` | No | scinote | Subfolder in bucket |
| `APPLICATION_URL` | No | http://localhost:37842 | Public URL |
| `WEB_PORT` | No | 37842 | Exposed web port |
| `POSTGRES_PORT` | No | 54329 | Exposed PostgreSQL port |
| `REDIS_PORT` | No | 63791 | Exposed Redis port |
| `DB_POOL` | No | 25 | Database connection pool |
| `RAILS_MAX_THREADS` | No | 5 | Rails threads per process |
| `WEB_CONCURRENCY` | No | 2 | Number of web processes |

\* Required for S3 storage functionality

### Email Configuration (Optional)

Configure SMTP for email notifications:

```env
MAIL_SERVER_URL=smtp.gmail.com
MAIL_SERVER_PORT=587
MAIL_SERVER_USERNAME=your_email@gmail.com
MAIL_SERVER_PASSWORD=your_app_password
MAIL_FROM=noreply@yourdomain.com
```

## 🔒 Security Best Practices

1. **Secret Key**: Generate a strong `SECRET_KEY_BASE`
   ```bash
   openssl rand -hex 64
   ```

2. **Database Password**: Use a strong, unique password

3. **S3 Bucket Security**:
   - Keep bucket private (block public access)
   - Use IAM roles with minimal permissions
   - Enable bucket encryption
   - Enable versioning for data protection

4. **HTTPS**: Use a reverse proxy (Nginx, Traefik) with SSL/TLS

5. **Firewall**: Restrict exposed ports to trusted IPs only
   - PostgreSQL (54329): Only allow from admin IPs for database management
   - Redis (63791): Only allow from admin IPs for cache management
   - Web (37842): Allow from all IPs or use reverse proxy
   - Consider removing port mappings for db and redis in production if external access is not needed

## 📊 Monitoring & Maintenance

### View Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f web
docker-compose logs -f jobs
```

### Health Checks

```bash
# Check service health
docker-compose ps

# Test database connection
docker-compose exec db pg_isready

# Test Redis connection
docker-compose exec redis redis-cli ping
```

### Database Backup

```bash
# Backup database (via Docker)
docker-compose exec db pg_dump -U postgres scinote_production > backup.sql

# Restore database (via Docker)
docker-compose exec -T db psql -U postgres scinote_production < backup.sql

# Backup database (via exposed port)
pg_dump -h localhost -p 54329 -U postgres scinote_production > backup.sql

# Restore database (via exposed port)
psql -h localhost -p 54329 -U postgres scinote_production < backup.sql
```

### Update SciNote

```bash
# Pull latest image
docker-compose pull web jobs

# Restart services
docker-compose up -d

# Run migrations
docker-compose exec web rake db:migrate
```

## 🐛 Troubleshooting

### Web Service Won't Start

1. Check logs: `docker-compose logs web`
2. Verify database is running: `docker-compose ps db`
3. Ensure `SECRET_KEY_BASE` is set
4. Check database connection settings

### S3 Upload Failures

1. Verify AWS credentials are correct
2. Check S3 bucket exists and is accessible
3. Verify IAM permissions include required S3 actions
4. Check `ACTIVESTORAGE_SERVICE` is set to `amazon`
5. Review logs: `docker-compose logs web | grep -i s3`

### Database Connection Issues

1. Check database is healthy: `docker-compose ps db`
2. Verify `DATABASE_URL` or individual DB variables
3. Ensure database has been initialized: `docker-compose exec web rake db:migrate`

### Performance Issues

1. Increase `DB_POOL` for more database connections
2. Increase `RAILS_MAX_THREADS` for more concurrent requests
3. Scale web workers: `docker-compose up -d --scale web=3`
4. Monitor resource usage: `docker stats`

## 📚 Additional Resources

### Documentation Files
- [PORT_CONFIGURATION.md](PORT_CONFIGURATION.md) - Detailed port configuration guide
- [easypanel-setup.md](easypanel-setup.md) - Easypanel deployment guide
- [s3-configuration-guide.md](s3-configuration-guide.md) - S3 storage setup
- [DEPLOYMENT_SUMMARY.md](DEPLOYMENT_SUMMARY.md) - Quick reference summary

### External Resources
- [SciNote Official Documentation](https://www.scinote.net/docs/)
- [SciNote GitHub Repository](https://github.com/scinote-eln/scinote-web)
- [AWS S3 Documentation](https://docs.aws.amazon.com/s3/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Easypanel Documentation](https://easypanel.io/docs)

## 🤝 Support

For issues and questions:
- SciNote Community: [community.scinote.net](https://community.scinote.net)
- GitHub Issues: [scinote-web/issues](https://github.com/scinote-eln/scinote-web/issues)

## 📄 License

SciNote is licensed under the MIT License. See the [LICENSE](https://github.com/scinote-eln/scinote-web/blob/develop/LICENSE) file for details.