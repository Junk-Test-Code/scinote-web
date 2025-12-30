# Port Configuration Guide

This document explains the port configuration for the SciNote Docker Compose deployment.

## 🔢 Port Assignments

All ports have been set to unique random values to avoid conflicts with other services running in the same Docker environment.

### Exposed Ports

| Service | Internal Port | External Port | Purpose |
|---------|--------------|---------------|---------|
| **Web Application** | 3000 | **37842** | Main SciNote web interface |
| **PostgreSQL** | 5432 | **54329** | Database access for management tools |
| **Redis** | 6379 | **63791** | Cache access for monitoring tools |

### Port Selection Rationale

The ports were chosen to be:
1. **Highly unique**: Random 5-digit numbers unlikely to conflict with common services
2. **Above 32768**: In the dynamic/private port range to avoid well-known ports
3. **Non-sequential**: Not following any pattern to maximize uniqueness

### Common Port Conflicts Avoided

These unique ports avoid conflicts with commonly used services:

| Service | Common Port | Our Port |
|---------|-------------|----------|
| Node.js/React dev | 3000 | 37842 |
| Python dev servers | 8000 | 37842 |
| HTTP alternate | 8080 | 37842 |
| PostgreSQL default | 5432 | 54329 |
| Redis default | 6379 | 63791 |
| Flask | 5000 | 37842 |
| Rails default | 3000 | 37842 |
| Webpack dev | 8080 | 37842 |
| Next.js | 3000 | 37842 |

## 🔧 Configuration

### Environment Variables

Set these in your `.env` file:

```env
# Web application port
WEB_PORT=37842

# PostgreSQL database port
POSTGRES_PORT=54329

# Redis cache port
REDIS_PORT=63791
```

### Accessing Services

#### Web Application
```bash
# Local access
http://localhost:37842

# Production (with domain)
https://your-domain.com
```

#### PostgreSQL Database
```bash
# Using psql
psql -h localhost -p 54329 -U postgres -d scinote_production

# Using connection string
postgresql://postgres:password@localhost:54329/scinote_production

# Using GUI tools (pgAdmin, DBeaver, etc.)
Host: localhost
Port: 54329
Database: scinote_production
Username: postgres
Password: <your_password>
```

#### Redis Cache
```bash
# Using redis-cli
redis-cli -h localhost -p 63791

# Check connection
redis-cli -h localhost -p 63791 ping
# Should return: PONG

# Monitor Redis
redis-cli -h localhost -p 63791 monitor
```

## 🔒 Security Considerations

### Production Deployment

For production environments, consider the following security measures:

#### 1. Remove Unnecessary Port Exposures

If you don't need external access to PostgreSQL or Redis, remove their port mappings from `docker-compose.yml`:

```yaml
# Remove these lines for db service:
ports:
  - "${POSTGRES_PORT:-54329}:5432"

# Remove these lines for redis service:
ports:
  - "${REDIS_PORT:-63791}:6379"
```

Services will still be accessible within the Docker network for the web and jobs containers.

#### 2. Firewall Rules

If you keep ports exposed, restrict access using firewall rules:

```bash
# Allow web access from anywhere
sudo ufw allow 37842/tcp

# Allow PostgreSQL only from specific IP
sudo ufw allow from 203.0.113.10 to any port 54329

# Allow Redis only from specific IP
sudo ufw allow from 203.0.113.10 to any port 63791
```

#### 3. Use Reverse Proxy

For the web application, use a reverse proxy (Nginx, Traefik, Caddy) with SSL:

```nginx
# Nginx example
server {
    listen 80;
    server_name scinote.yourdomain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name scinote.yourdomain.com;
    
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    location / {
        proxy_pass http://localhost:37842;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

#### 4. SSH Tunneling

For secure remote database access, use SSH tunneling instead of exposing ports:

```bash
# Create SSH tunnel for PostgreSQL
ssh -L 54329:localhost:54329 user@your-server.com

# Now connect to localhost:54329 on your local machine
psql -h localhost -p 54329 -U postgres -d scinote_production
```

## 🔍 Monitoring

### Check Port Usage

```bash
# Check if ports are in use
netstat -tuln | grep -E '37842|54329|63791'

# Or using ss
ss -tuln | grep -E '37842|54329|63791'

# Check what's listening on specific port
lsof -i :37842
lsof -i :54329
lsof -i :63791
```

### Docker Port Mapping

```bash
# List all port mappings
docker-compose ps

# Check specific service ports
docker port scinote_web
docker port scinote_db
docker port scinote_redis
```

## 🛠️ Troubleshooting

### Port Already in Use

If you get "port already in use" errors:

1. **Check what's using the port:**
   ```bash
   lsof -i :37842
   ```

2. **Change the port in .env:**
   ```env
   WEB_PORT=38000  # Use a different port
   ```

3. **Restart services:**
   ```bash
   docker-compose down
   docker-compose up -d
   ```

### Cannot Connect to Database

1. **Verify port is exposed:**
   ```bash
   docker port scinote_db
   ```

2. **Check database is running:**
   ```bash
   docker-compose ps db
   ```

3. **Test connection:**
   ```bash
   psql -h localhost -p 54329 -U postgres -d scinote_production
   ```

### Cannot Connect to Redis

1. **Verify port is exposed:**
   ```bash
   docker port scinote_redis
   ```

2. **Check Redis is running:**
   ```bash
   docker-compose ps redis
   ```

3. **Test connection:**
   ```bash
   redis-cli -h localhost -p 63791 ping
   ```

## 📝 Customization

### Changing Ports

To use different ports:

1. **Edit .env file:**
   ```env
   WEB_PORT=40000
   POSTGRES_PORT=50000
   REDIS_PORT=60000
   ```

2. **Restart services:**
   ```bash
   docker-compose down
   docker-compose up -d
   ```

3. **Update firewall rules** if applicable

4. **Update reverse proxy configuration** if applicable

### Using Standard Ports

If you want to use standard ports (not recommended for shared environments):

```env
WEB_PORT=3000
POSTGRES_PORT=5432
REDIS_PORT=6379
```

**Warning**: This may conflict with other services using these common ports.

## 🌐 Easypanel Considerations

When deploying on Easypanel:

1. **Port Mapping**: Easypanel handles port mapping automatically
2. **Domain Routing**: Use Easypanel's domain configuration instead of direct port access
3. **Internal Services**: Database and Redis should remain internal to the Docker network
4. **SSL/TLS**: Easypanel provides automatic SSL certificate management

### Easypanel Port Configuration

In Easypanel, you typically:
- **Don't expose** database and Redis ports externally
- **Use domain names** instead of IP:PORT combinations
- **Let Easypanel handle** SSL/TLS termination
- **Configure** only the web service port (37842)

## ✅ Best Practices

1. **Use unique ports** to avoid conflicts in shared environments
2. **Restrict access** to database and Redis ports using firewall rules
3. **Use SSH tunneling** for remote database access instead of exposing ports
4. **Enable SSL/TLS** for web traffic using reverse proxy or Easypanel
5. **Monitor port usage** regularly to detect unauthorized access
6. **Document changes** when modifying port configurations
7. **Test connectivity** after any port changes

---

For more information, see:
- [README.md](README.md) - General deployment guide
- [easypanel-setup.md](easypanel-setup.md) - Easypanel-specific instructions
- [.env.example](.env.example) - Environment variable reference