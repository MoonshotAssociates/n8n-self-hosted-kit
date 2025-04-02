# n8n Self-Hosted Kit

A complete self-hosted solution for running n8n workflow automation in a production environment with Docker Compose.

## Overview

This kit provides a scalable n8n setup with:

- Main n8n instance
- Multiple worker nodes for distributed execution
- PostgreSQL database for data persistence
- Redis for queue management
- Nginx Proxy Manager for SSL/TLS and routing

## Prerequisites

- Docker and Docker Compose installed
- At least 4GB RAM recommended
- Storage volumes mounted at `/storage/` paths

## Quick Start

1. Clone this repository
2. Copy `.env.sample` to `.env` and configure your environment variables (see Configuration section)
3. Start the services:

```bash
sudo docker compose up -d
```

4. Access n8n at http://localhost:5678
5. Access Nginx Proxy Manager admin at http://localhost:81 (default credentials: admin@example.com / changeme)

## Configuration

The system is configured via environment variables in the `.env` file:

```
# postgres environment variables
POSTGRES_USER=n8n
POSTGRES_PASSWORD=change-to-secure-password
POSTGRES_DB=n8n
POSTGRES_HOST_AUTH_METHOD=md5

# n8n environment variables
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=postgres
DB_POSTGRESDB_PORT=5432
DB_POSTGRESDB_DATABASE=n8n
DB_POSTGRESDB_USER=n8n
DB_POSTGRESDB_PASSWORD=change-to-secure-password

# n8n Queue Configuration
QUEUE_MODE_ENABLED=true
QUEUE_BULL_REDIS_HOST=redis
QUEUE_BULL_REDIS_PORT=6379
EXECUTIONS_MODE=queue
# Note: EXECUTIONS_PROCESS has been deprecated and removed

# n8n Security
# IMPORTANT: Generate a secure random string for production use
N8N_ENCRYPTION_KEY=change-to-secure-encryption-key

# Enforce proper permissions on settings file (recommended)
N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
N8N_LOG_LEVEL=debug
N8N_PORT=5678
N8N_PROTOCOL=http
N8N_HOST=0.0.0.0
NODE_FUNCTION_ALLOW_EXTERNAL=true

# n8n Webhook URL
# Set this to your public-facing domain where n8n will be accessible
WEBHOOK_URL=https://your-domain.com/

# Redis Configuration
REDIS_HOST=redis
REDIS_PORT=6379

# Optional: Uncomment and set these for additional configuration
# N8N_METRICS=true
# N8N_METRICS_PORT=9100
# N8N_DIAGNOSTICS_ENABLED=false
# N8N_DIAGNOSTICS_CONFIG={"enabled": false}
```

**Important**: Replace `strong-password`, `strong-key`, and update the `WEBHOOK_URL` with your actual domain in production.

## Architecture

- **n8n-main**: Primary n8n instance that handles the UI and workflow management
- **n8n-worker-1/2/3**: Worker nodes that process workflow executions from the queue
- **postgres**: Database for storing workflows, credentials, and execution data
- **redis**: Queue management for distributing work to the workers
- **nginx**: Reverse proxy for handling SSL/TLS and routing

## Storage Volumes

The setup uses Docker volumes with bind mounts to persistent storage:

- `/storage/postgres_data`: PostgreSQL data
- `/storage/n8n_storage`: n8n configuration and data
- `/storage/n8n_backup`: Backup location for n8n
- `/storage/n8n_shared`: Shared storage between n8n instances
- `/storage/letsencrypt`: SSL certificates and storage
- `/storage/nginx_data`: Nginx configuration

## Docker Compose Configuration

The `docker-compose.yml` file defines the following services:

1. **postgres**: PostgreSQL database using the latest image
   - Uses environment variables from `.env`
   - Health checks to ensure it's ready before dependent services start

2. **nginx**: Nginx Proxy Manager for handling SSL/TLS and routing
   - Exposes ports 80, 81 (admin), and 443
   - Mounts volumes for certificates and configuration

3. **redis**: Redis for queue management
   - Health checks to ensure it's ready

4. **n8n-main**: Main n8n instance
   - Uses environment variables from `.env`
   - Exposes port 5678 for web UI
   - Depends on postgres and redis being healthy

5. **n8n-worker-1/2/3**: Worker nodes
   - Use environment variables from `.env`
   - Depend on postgres, redis, and n8n-main
   - Run in worker mode to process executions from the queue

## Scaling

To add more worker nodes, duplicate the worker service definition in `docker-compose.yml` and increment the container name.

## Maintenance

### Backups

Backups can be stored in the `/storage/n8n_backup` directory. Set up a cron job to regularly export your workflows.

### Updates

To update n8n to the latest version:

```bash
sudo docker compose pull
sudo docker compose up -d
```

## Troubleshooting

- Check logs: `sudo docker compose logs -f [service_name]`
- Validate configuration: `sudo docker compose config`
- Restart services: `sudo docker compose restart [service_name]`

## Security Considerations

- Change default passwords in the `.env` file
- Use strong encryption keys
- Configure Nginx Proxy Manager with proper SSL/TLS settings
- Restrict access to the n8n admin interface
