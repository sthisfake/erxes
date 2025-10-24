# Quick Start Guide - Deploy erxes with Docker (No SSL)

## Step 1: Create Environment File

Create a `.env` file in the project root:

```bash
# Copy the sample
cp .env.sample .env
```

Or create `.env` manually with this content:

```env
# Domain Configuration (use your server IP)
DOMAIN=http://YOUR_SERVER_IP:3001
ALLOWED_DOMAINS=http://YOUR_SERVER_IP:3001

# Node Configuration
NODE_ENV=production
NODE_OPTIONS=--max-old-space-size=1536

# Optional: Enable specific plugins
ENABLED_PLUGINS=operation,frontline,sales,payment
```

**Replace `YOUR_SERVER_IP` with your actual server IP address** (e.g., `http://192.168.1.100:3001`)

## Step 2: Build Docker Images

```bash
docker-compose build
```

This will take 10-20 minutes depending on your server.

## Step 3: Start All Services

```bash
docker-compose up -d
```

## Step 4: Check Services Status

```bash
docker-compose ps
```

All services should show "Up" status.

## Step 5: View Logs (Optional)

```bash
# View all logs
docker-compose logs -f

# View specific service logs
docker-compose logs -f gateway
docker-compose logs -f frontend
```

## Access Your Application

- **Frontend**: `http://YOUR_SERVER_IP:3001`
- **API Gateway**: `http://YOUR_SERVER_IP:4000/graphql`
- **RabbitMQ Management**: `http://YOUR_SERVER_IP:15672` (username: `erxes`, password: `erxes123`)

## Common Commands

### Stop All Services
```bash
docker-compose down
```

### Restart All Services
```bash
docker-compose restart
```

### Restart Specific Service
```bash
docker-compose restart gateway
```

### View Service Logs
```bash
docker-compose logs -f [service-name]
```

### Rebuild and Restart
```bash
docker-compose down
docker-compose build --no-cache
docker-compose up -d
```

## Troubleshooting

### If services fail to start:

1. **Check logs**:
   ```bash
   docker-compose logs [service-name]
   ```

2. **Check if ports are available**:
   ```bash
   netstat -tulpn | grep -E '3001|3300|4000|27017|6379'
   ```

3. **Restart specific service**:
   ```bash
   docker-compose restart [service-name]
   ```

### If you need to clean everything:

```bash
# Stop and remove containers
docker-compose down

# Remove all volumes (WARNING: This deletes all data!)
docker-compose down -v

# Rebuild from scratch
docker-compose build --no-cache
docker-compose up -d
```

## Server Firewall Configuration

Make sure these ports are open on your server:

```bash
# For Ubuntu/Debian with UFW
sudo ufw allow 3001/tcp  # Frontend
sudo ufw allow 4000/tcp  # Gateway API
sudo ufw allow 15672/tcp # RabbitMQ Management (optional)

# For CentOS/RHEL with firewalld
sudo firewall-cmd --permanent --add-port=3001/tcp
sudo firewall-cmd --permanent --add-port=4000/tcp
sudo firewall-cmd --permanent --add-port=15672/tcp
sudo firewall-cmd --reload
```

## That's It!

Your erxes platform should now be running at `http://YOUR_SERVER_IP:3001`
