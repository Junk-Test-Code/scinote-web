# Easypanel Deployment Fix

## Issue Fixed
The original Docker Compose configuration was trying to build the SciNote web application from source code using a build context that doesn't exist in the Easypanel environment. This caused the error:

```
unable to prepare context: path "/etc/easypanel/projects/personal/scinote-test/code/scinote-web" not found
```

Additionally, Easypanel was creating a `docker-compose.override.yml` file that was trying to expose port 3000 externally, causing port conflicts:

```
Bind for 0.0.0.0:3000 failed: port is already allocated
```

## Solution
Changed both `web` and `jobs` services to use the official pre-built Docker image from Docker Hub and added an override file to prevent port conflicts.

### Before:
```yaml
web:
  build:
    context: ./scinote-web
    dockerfile: Dockerfile.production

jobs:
  build:
    context: ./scinote-web
    dockerfile: Dockerfile.production
```

### After:
```yaml
web:
  image: scinote/scinote-web:latest

jobs:
  image: scinote/scinote-web:latest
```

### Port Configuration
Added `docker-compose.override.yml` to prevent Easypanel from creating conflicting port mappings:

- **Web Application**: External port `37842` → Internal port `3000`
- **PostgreSQL**: External port `54329` → Internal port `5432`
- **Redis**: External port `63791` → Internal port `6379`
- **Jobs Service**: No external ports (background worker only)

The override file ensures that Easypanel cannot override these port settings with default values like port 3000, which would cause conflicts.

## Benefits of Using Pre-built Images
1. **No Build Required**: Eliminates build timeouts and complex dependencies
2. **Faster Deployment**: Pull images directly from Docker Hub
3. **Stable Version**: Uses officially tested and released images
4. **Reduced Storage**: No need for source code on the server
5. **Easier Updates**: Simply change the image tag to update versions
6. **No Port Conflicts**: Custom ports prevent collisions with other services

## Deployment Steps with Easypanel

### 1. Update Your Repository
The fixed `docker-compose.yml` and `docker-compose.override.yml` have been updated. You can now:

```bash
# Commit the changes
git add docker-compose.yml docker-compose.override.yml
git commit -m "Fix: Use pre-built Docker images and prevent port conflicts"

# Push to your repository
git push origin feature/docker-compose-fix
```

### 2. Redeploy in Easypanel
After pushing the changes, go to your Easypanel project and:

1. Pull the latest changes from your repository
2. Redeploy the service
3. The deployment should now succeed without build or port errors

### 3. Verify Deployment
Check that all services are running:
- PostgreSQL database (port 54329)
- Redis cache (port 63791)
- SciNote web application (port 37842)
- Background jobs worker

### 4. Access SciNote
Once deployed, access SciNote at:
```
http://your-server-ip:37842
```

## Configuration Requirements

Make sure your `.env` file contains at minimum:

```bash
# Required
SECRET_KEY_BASE=your_secret_key_here
S3_BUCKET=your_bucket_name
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key

# Optional - for S3-compatible services
AWS_ENDPOINT=https://your-s3-endpoint.com
AWS_FORCE_PATH_STYLE=true
```

## Troubleshooting

### Port Conflicts
If you still encounter port conflicts:
1. Check what services are using ports 37842, 54329, 63791
2. Use `docker ps` to see running containers and their port mappings
3. Adjust the port numbers in both `docker-compose.yml` and `docker-compose.override.yml`

### Image Pull Issues
If you encounter issues pulling the image, ensure:
- Docker Hub is accessible from your server
- You have sufficient disk space
- Your internet connection is stable

### Database Connection
If the web service can't connect to the database:
- Verify the database service is healthy
- Check the environment variables match
- Ensure both services are on the same network

### S3 Configuration
For S3-compatible services (DigitalOcean, Wasabi, MinIO, etc.):
- Set `AWS_ENDPOINT` to your service's endpoint URL
- Set `AWS_FORCE_PATH_STYLE=true`
- Ensure your bucket exists and credentials are correct

## Support
For issues with the deployment:
1. Check Easypanel logs for detailed error messages
2. Verify all environment variables are set correctly
3. Ensure S3 credentials and bucket configuration are valid
4. Review the full documentation in README.md and s3-configuration-guide.md