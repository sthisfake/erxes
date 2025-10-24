# erxes Docker Deployment Guide

## Architecture Overview

### Services
- **Core API** (3300): Main API service
- **Gateway** (4000): GraphQL API Gateway
- **Automations** (3400): Automation workflows
- **Logs** (3500): Logging service
- **Operation API** (4010): Operation plugin
- **Frontline API** (4011): Frontline plugin
- **Sales API** (4012): Sales plugin
- **Payment API** (4013): Payment plugin
- **Frontend** (3001): React UI

### Infrastructure
- **MongoDB** (27017): Database
- **Redis** (6379): Cache
- **RabbitMQ** (5672, 15672): Message queue
- **Elasticsearch** (9200): Search

## Prerequisites

- Docker 20.10+
- Docker Compose 2.0+
- 8GB RAM minimum
- 20GB disk space

## Quick Start

### 1. Clone and Configure

```bash
cd /path/to/erxes
cp .env.sample .env
```

### 2. Edit Environment Variables

Create `.env` file:

```env
# Domain Configuration
DOMAIN=http://localhost:3001
ALLOWED_DOMAINS=http://localhost:3001

# Optional: Client Portal
CLIENT_PORTAL_DOMAINS=

# Node Configuration
NODE_ENV=production
NODE_OPTIONS=--max-old-space-size=1536
```

### 3. Build and Start

```bash
# Build all services
docker-compose build

# Start all services
docker-compose up -d

# View logs
docker-compose logs -f
```

### 4. Access Application

- **Frontend**: http://localhost:3001
- **API Gateway**: http://localhost:4000/graphql
- **RabbitMQ Management**: http://localhost:15672 (user: erxes, pass: erxes123)

## Production Deployment

### 1. Server Setup

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### 2. Configure Production Environment

```env
DOMAIN=https://yourdomain.com
ALLOWED_DOMAINS=https://yourdomain.com
NODE_ENV=production
```

### 3. Deploy with SSL (Recommended)

Add nginx reverse proxy with SSL:

```yaml
# Add to docker-compose.yml
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx-ssl.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - frontend
      - gateway
```

### 4. Start Production

```bash
docker-compose up -d
```

## Service Management

### Start Services
```bash
docker-compose up -d
```

### Stop Services
```bash
docker-compose down
```

### Restart Specific Service
```bash
docker-compose restart gateway
```

### View Logs
```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f gateway
```

### Scale Services
```bash
# Scale gateway to 3 instances
docker-compose up -d --scale gateway=3
```

## Monitoring

### Check Service Health
```bash
docker-compose ps
```

### Resource Usage
```bash
docker stats
```

### Database Connection
```bash
docker exec -it erxes-mongodb mongosh erxes
```

### Redis Connection
```bash
docker exec -it erxes-redis redis-cli
```

## Backup and Restore

### Backup MongoDB
```bash
# Create backup
docker exec erxes-mongodb mongodump --out=/backup --db=erxes

# Copy to host
docker cp erxes-mongodb:/backup ./mongodb-backup-$(date +%Y%m%d)
```

### Restore MongoDB
```bash
# Copy backup to container
docker cp ./mongodb-backup erxes-mongodb:/backup

# Restore
docker exec erxes-mongodb mongorestore /backup
```

### Backup Volumes
```bash
# Stop services
docker-compose down

# Backup volumes
docker run --rm -v erxes_mongodb_data:/data -v $(pwd):/backup alpine tar czf /backup/mongodb-data.tar.gz /data

# Start services
docker-compose up -d
```

## Troubleshooting

### Service Won't Start

```bash
# Check logs
docker-compose logs [service-name]

# Check if port is in use
netstat -tulpn | grep [port]

# Rebuild service
docker-compose build --no-cache [service-name]
docker-compose up -d [service-name]
```

### Database Connection Issues

```bash
# Check MongoDB status
docker exec erxes-mongodb mongosh --eval "db.adminCommand('ping')"

# Check Redis status
docker exec erxes-redis redis-cli ping
```

### Memory Issues

Edit `.env`:
```env
NODE_OPTIONS=--max-old-space-size=2048
```

Then restart:
```bash
docker-compose restart
```

### Clean Everything and Restart

```bash
# Stop and remove containers
docker-compose down

# Remove volumes (WARNING: deletes all data)
docker-compose down -v

# Remove images
docker-compose down --rmi all

# Rebuild and start
docker-compose build --no-cache
docker-compose up -d
```

## Performance Optimization

### 1. Enable Production Mode
Ensure `NODE_ENV=production` in `.env`

### 2. Increase Memory Limits
```yaml
# In docker-compose.yml, add to services:
    deploy:
      resources:
        limits:
          memory: 2G
        reservations:
          memory: 1G
```

### 3. Use External Databases
For production, consider using managed services:
- MongoDB Atlas
- Redis Cloud
- Amazon RDS

Update connection strings in `.env`

## Security Best Practices

1. **Change Default Passwords**
   - Update RabbitMQ credentials in docker-compose.yml
   - Set strong MongoDB passwords

2. **Use SSL/TLS**
   - Configure nginx with SSL certificates
   - Use Let's Encrypt for free SSL

3. **Firewall Configuration**
   ```bash
   # Allow only necessary ports
   sudo ufw allow 80/tcp
   sudo ufw allow 443/tcp
   sudo ufw enable
   ```

4. **Regular Updates**
   ```bash
   docker-compose pull
   docker-compose up -d
   ```

## Advanced Configuration

### Custom Plugin Ports

Edit `docker-compose.yml` to add more plugins:

```yaml
  custom-plugin-api:
    build:
      context: .
      dockerfile: Dockerfile.backend
      target: plugins
    working_dir: /app/backend/plugins/custom_api
    command: ["pnpm", "start"]
    environment:
      PORT: 4020
      MONGO_URL: mongodb://mongodb:27017/erxes
      REDIS_HOST: redis
    ports:
      - "4020:4020"
```

### Environment-Specific Configs

Create multiple compose files:
- `docker-compose.yml` (base)
- `docker-compose.prod.yml` (production overrides)
- `docker-compose.dev.yml` (development overrides)

Use:
```bash
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

## Support

For issues and questions:
- GitHub: https://github.com/erxes/erxes
- Discord: https://discord.com/invite/aaGzy3gQK5
- Documentation: https://erxes.io/docs
