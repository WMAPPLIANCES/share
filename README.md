# share# Chatwoot + Supabase Docker Setup

![Chatwoot](https://raw.githubusercontent.com/chatwoot/chatwoot/develop/public/brand-assets/logo.svg)

> **Complete setup for Chatwoot v4 with Supabase integration using Docker Swarm**

## 📋 Table of Contents

- [Features](#-features)
- [Prerequisites](#-prerequisites)
- [Quick Start](#-quick-start)
- [Configuration](#-configuration)
- [Deployment](#-deployment)
- [Management](#-management)
- [Troubleshooting](#-troubleshooting)
- [Contributing](#-contributing)

## ✨ Features

- 🐳 **Docker Swarm** ready configuration
- 🗄️ **Supabase** integration with pgvector support
- 🔒 **SSL/HTTPS** with Traefik reverse proxy
- 📧 **SMTP** email configuration
- 📊 **Log rotation** (10MB max, 3 files)
- 🌍 **Multi-language** support (en_US default)
- ⚡ **Performance** optimized with Sidekiq
- 📁 **Persistent storage** with Docker volumes

## 🔧 Prerequisites

Before starting, ensure you have:

- [x] Docker Engine 20.10+ with Swarm mode enabled
- [x] Supabase running locally with pgvector extension
- [x] Traefik configured for SSL termination
- [x] Domain pointing to your server

### Enable Docker Swarm

```bash
docker swarm init
```

### Required Extensions in Supabase

```sql
-- Connect to your Supabase and run:
CREATE EXTENSION IF NOT EXISTS vector;
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
```

## 🚀 Quick Start

### 1. Clone and Setup

```bash
# Create project directory
mkdir chatwoot-setup
cd chatwoot-setup

# Download the docker-compose.yml
# (paste the compose file content here)
```

### 2. Create Required Volumes

```bash
docker volume create chatwoot_storage
docker volume create chatwoot_public
docker volume create chatwoot_mailer
docker volume create chatwoot_mailers
```

### 3. Create Network

```bash
docker network create --driver overlay agent_network
```

### 4. Configure Environment

Edit the `docker-compose.yml` file and replace these variables:

```bash
# Required Changes
INSTALLATION_NAME=your_company_name
SECRET_KEY_BASE=your_secret_key_here
FRONTEND_URL=https://your-chatwoot-domain.com
POSTGRES_PASSWORD=your_supabase_password

# SMTP Configuration
MAILER_SENDER_EMAIL=your_email@domain.com
SMTP_DOMAIN=your-domain.com
SMTP_USERNAME=your_smtp_username
SMTP_PASSWORD=your_smtp_password
```

### 5. Deploy

```bash
docker stack deploy -c docker-compose.yml chatwoot
```

## ⚙️ Configuration

### Environment Variables Reference

| Category | Variable | Description | Default |
|----------|----------|-------------|---------|
| **App** | `INSTALLATION_NAME` | Company/installation name | `your_company_name` |
| **Security** | `SECRET_KEY_BASE` | Rails secret key | Generate with `rails secret` |
| **URL** | `FRONTEND_URL` | Chatwoot public URL | `https://your-domain.com` |
| **Database** | `POSTGRES_HOST` | Supabase host | `db` |
| **Database** | `POSTGRES_USERNAME` | Supabase user | `supabase_admin` |
| **Database** | `POSTGRES_PASSWORD` | Supabase password | Your password |
| **Database** | `POSTGRES_DATABASE` | Database name | `postgres` |
| **Email** | `SMTP_ADDRESS` | SMTP server | `smtp.gmail.com` |
| **Email** | `SMTP_PORT` | SMTP port | `587` |
| **Email** | `SMTP_USERNAME` | SMTP username | Your username |
| **Email** | `SMTP_PASSWORD` | SMTP password | Your password |
| **Locale** | `DEFAULT_LOCALE` | Default language | `en_US` |
| **Timezone** | `TZ` | Server timezone | `America/New_York` |

### Generate Secret Key

```bash
# Method 1: Using Ruby
ruby -e "require 'securerandom'; puts SecureRandom.hex(64)"

# Method 2: Using OpenSSL
openssl rand -hex 64

# Method 3: Using Docker
docker run --rm ruby:3.1 ruby -e "require 'securerandom'; puts SecureRandom.hex(64)"
```

### SMTP Providers Examples

#### Gmail
```yaml
SMTP_ADDRESS=smtp.gmail.com
SMTP_PORT=587
SMTP_SSL=false
SMTP_USERNAME=your_gmail@gmail.com
SMTP_PASSWORD=your_app_password
```

#### SendGrid
```yaml
SMTP_ADDRESS=smtp.sendgrid.net
SMTP_PORT=587
SMTP_USERNAME=apikey
SMTP_PASSWORD=your_sendgrid_api_key
```

#### Mailgun
```yaml
SMTP_ADDRESS=smtp.mailgun.org
SMTP_PORT=587
SMTP_USERNAME=your_mailgun_username
SMTP_PASSWORD=your_mailgun_password
```

## 🚢 Deployment

### Deploy to Swarm

```bash
# Deploy stack
docker stack deploy -c docker-compose.yml chatwoot

# Check services status
docker service ls | grep chatwoot

# Watch deployment progress
watch docker service ls
```

### Update Configuration

```bash
# After changing docker-compose.yml
docker stack deploy -c docker-compose.yml chatwoot

# Force update without changes
docker service update --force chatwoot_chatwoot_app
```

### Scale Services

```bash
# Scale web app (if needed)
docker service scale chatwoot_chatwoot_app=2

# Scale sidekiq workers
docker service scale chatwoot_chatwoot_sidekiq=3
```

## 🛠️ Management

### Service Commands

```bash
# View service logs
docker service logs -f chatwoot_chatwoot_app
docker service logs -f chatwoot_chatwoot_sidekiq

# Inspect service
docker service inspect chatwoot_chatwoot_app

# List running containers
docker service ps chatwoot_chatwoot_app
```

### Database Operations

```bash
# Connect to Rails console
docker exec -it $(docker ps -q -f name=chatwoot_app) rails console

# Run database migrations
docker exec -it $(docker ps -q -f name=chatwoot_app) rails db:migrate

# Create admin user
docker exec -it $(docker ps -q -f name=chatwoot_app) rails console
# In console: User.create!(name: 'Admin', email: 'admin@example.com', password: 'password', role: 'administrator')
```

### Backup & Restore

```bash
# Backup Supabase database
docker exec -t $(docker ps -q -f name=supabase) pg_dump -U supabase_admin postgres > chatwoot_backup_$(date +%Y%m%d).sql

# Backup volumes
docker run --rm -v chatwoot_storage:/data -v $(pwd):/backup alpine tar czf /backup/chatwoot_storage_backup.tar.gz -C /data .

# Restore database
docker exec -i $(docker ps -q -f name=supabase) psql -U supabase_admin postgres < chatwoot_backup.sql
```

### Log Management

```bash
# View live logs
docker service logs -f chatwoot_chatwoot_app

# Check log file sizes (logs are auto-rotated)
docker exec $(docker ps -q -f name=chatwoot_app) du -sh /var/lib/docker/containers/*/
```

## 🔍 Troubleshooting

### Common Issues

#### Database Connection Failed
```bash
# Check if Supabase is running
docker ps | grep supabase

# Test connection
docker exec -it $(docker ps -q -f name=chatwoot_app) bash
# Inside container: 
psql -h db -U supabase_admin -d postgres
```

#### SSL/HTTPS Issues
```bash
# Check Traefik labels
docker service inspect chatwoot_chatwoot_app | grep -A 20 Labels

# Verify domain resolution
nslookup your-chatwoot-domain.com

# Check Traefik dashboard
curl -k https://your-traefik-dashboard.com/api/http/routers
```

#### Application Won't Start
```bash
# Check initialization logs
docker service logs chatwoot_chatwoot_db_init

# Verify all required environment variables
docker service inspect chatwoot_chatwoot_app | grep -A 50 Env

# Check if migrations completed
docker exec -it $(docker ps -q -f name=chatwoot_app) rails db:version
```

#### Performance Issues
```bash
# Monitor resource usage
docker stats

# Check Sidekiq queue
docker exec -it $(docker ps -q -f name=chatwoot_app) rails console
# In console: Sidekiq::Queue.new.size

# Scale services if needed
docker service scale chatwoot_chatwoot_sidekiq=3
```

### Health Checks

```bash
# Application health
curl -k https://your-chatwoot-domain.com/health

# Database health
docker exec $(docker ps -q -f name=chatwoot_app) rails runner "puts ActiveRecord::Base.connection.active?"

# Redis health
docker exec $(docker ps -q -f name=redis) redis-cli ping
```

### Log Analysis

```bash
# Search for errors in logs
docker service logs chatwoot_chatwoot_app 2>&1 | grep -i error

# Monitor real-time errors
docker service logs -f chatwoot_chatwoot_app 2>&1 | grep -i "error\|exception\|failed"

# Check Sidekiq worker status
docker service logs chatwoot_chatwoot_sidekiq 2>&1 | grep -i "sidekiq"
```

## 📊 Monitoring

### Service Status Dashboard

```bash
#!/bin/bash
# chatwoot-status.sh
echo "=== Chatwoot Services Status ==="
docker service ls | grep chatwoot
echo ""
echo "=== Resource Usage ==="
docker stats --no-stream --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"
echo ""
echo "=== Recent Logs ==="
docker service logs --tail 10 chatwoot_chatwoot_app
```

### Performance Metrics

```bash
# Check database performance
docker exec $(docker ps -q -f name=supabase) psql -U supabase_admin postgres -c "
SELECT query, calls, total_time, mean_time 
FROM pg_stat_statements 
ORDER BY total_time DESC 
LIMIT 10;"

# Monitor queue lengths
docker exec $(docker ps -q -f name=chatwoot_app) rails runner "
puts 'Sidekiq Queues:'
Sidekiq::Queue.all.each { |q| puts \"#{q.name}: #{q.size}\" }
"
```

## 🔄 Updates

### Update Chatwoot

```bash
# Pull latest image
docker pull chatwoot/chatwoot:latest

# Update services
docker service update --image chatwoot/chatwoot:latest chatwoot_chatwoot_app
docker service update --image chatwoot/chatwoot:latest chatwoot_chatwoot_sidekiq
docker service update --image chatwoot/chatwoot:latest chatwoot_chatwoot_db_init

# Run any pending migrations
docker exec -it $(docker ps -q -f name=chatwoot_app) rails db:migrate
```

### Rollback

```bash
# Rollback to previous version
docker service rollback chatwoot_chatwoot_app
docker service rollback chatwoot_chatwoot_sidekiq
```

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🆘 Support

- 📖 [Chatwoot Documentation](https://www.chatwoot.com/docs)
- 🗄️ [Supabase Documentation](https://supabase.com/docs)
- 🐳 [Docker Swarm Documentation](https://docs.docker.com/engine/swarm/)
- 🔧 [Traefik Documentation](https://doc.traefik.io/traefik/)

## 🙏 Acknowledgments

- [Chatwoot Team](https://github.com/chatwoot/chatwoot) for the amazing open-source platform
- [Supabase Team](https://github.com/supabase/supabase) for the excellent PostgreSQL alternative
- Community contributors and testers

---

**⭐ If this helped you, please consider giving it a star!**
