# Bugsink Coolify Deployment

Docker Compose configuration for deploying Bugsink on Coolify.

## Features

- **Fixed ALLOWED_HOSTS**: Properly configured to work with Coolify's routing
- **MySQL Database**: Persistent data storage with health checks
- **Environment Variables**: Uses Coolify's variable system
- **Health Checks**: Automatic service health monitoring

## Deployment on Coolify

### 1. Connect GitHub Repository

1. In Coolify dashboard, go to **Sources**
2. Add your GitHub account if not already connected
3. Authorize Coolify to access your repositories

### 2. Create New Resource

1. Click **+ New Resource**
2. Select **Docker Compose**
3. Choose **GitHub** as the source
4. Select this repository: `gvocale/bugsink-coolify`
5. Branch: `main`
6. Compose File Path: `docker-compose.yml`

### 3. Configure Environment Variables

Coolify will auto-detect these variables from the compose file. Set them in the Coolify UI:

```env
SERVICE_PASSWORD_ROOT=<your-secure-root-password>
SERVICE_USER_BUGSINK=<your-database-user>
SERVICE_PASSWORD_BUGSINK=<your-secure-password>
SERVICE_PASSWORD_64_BUGSINK=<your-django-secret-key>
```

**Generate secure values:**

```bash
# For passwords
openssl rand -base64 32

# For Django SECRET_KEY (SERVICE_PASSWORD_64_BUGSINK)
openssl rand -base64 50
```

### 4. Configure Domain

Set your domain in Coolify, e.g.:

- `bugsink.yourdomain.com`
- Or use Coolify's auto-generated domain

**Important:** Update `BASE_URL` in `docker-compose.yml` to match your domain:

```yaml
BASE_URL: http://your-actual-domain.com
```

### 5. Deploy

1. Click **Deploy**
2. Wait for both services to become healthy (check logs)
3. Access your Bugsink instance at the configured domain

### 6. Initial Login

- Username: `admin`
- Password: The value of `SERVICE_PASSWORD_BUGSINK`

## Troubleshooting

### ALLOWED_HOSTS Error

If you see:

```
Host 'xxx' not in ALLOWED_HOSTS
```

**Solution:** Ensure `BASE_URL` in `docker-compose.yml` matches your actual domain (without port numbers).

### Database Connection Issues

Check that:

- MySQL service is healthy (green in Coolify)
- Environment variables are correctly set
- Database credentials match in both services

### Service Won't Start

1. Check Coolify logs for both services
2. Verify all environment variables are set
3. Ensure the domain is correctly configured

## Updates

To update your deployment:

1. Make changes to `docker-compose.yml`
2. Commit and push to GitHub
3. Coolify will auto-deploy (if enabled) or manually trigger deployment

## Architecture

```
┌─────────────────┐
│   Coolify       │
│   (Traefik)     │
└────────┬────────┘
         │
    ┌────▼─────┐
    │   Web    │ (Bugsink)
    │  :8000   │
    └────┬─────┘
         │
    ┌────▼─────┐
    │  MySQL   │
    │  :3306   │
    └──────────┘
```

## Support

For issues specific to:

- **Bugsink**: https://github.com/bugsink/bugsink
- **Coolify**: https://coolify.io/docs
- **This Config**: https://github.com/gvocale/bugsink-coolify/issues
