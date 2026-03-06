# Docs (on-premises) installation

## Overview

ONLYOFFICE Docs (formerly Document Server) is a self-hosted office suite that allows you to edit documents directly in your browser. Installing it on-premises gives you full control over your document editing environment and is ideal for testing plugins in a production-like setup.

## Why use ONLYOFFICE Docs for plugin development?

- **Production environment** - Test plugins in a real server setup
- **Multi-user testing** - Test collaborative features
- **Custom configuration** - Full control over server settings
- **API integration** - Test plugins with your own applications
- **Scalability testing** - Test plugin performance under load

## Prerequisites

Before installation, ensure you have:

- **Server or VM** with one of:
  - Ubuntu 20.04/22.04 LTS
  - Debian 10/11
  - CentOS 7/8
  - Red Hat Enterprise Linux 7/8
- **Minimum 4 GB RAM** (8 GB recommended)
- **Minimum 40 GB disk space**
- **Root or sudo access**
- **Domain name** (optional but recommended for HTTPS)

## Installation methods

ONLYOFFICE Docs can be installed using:

1. **Docker** (recommended for development)
2. **Package installation** (DEB/RPM)
3. **Docker Compose** (for custom setups)

## Method 1: Docker installation (recommended)

Docker provides the easiest and fastest way to get ONLYOFFICE Docs running.

### Install Docker

**Ubuntu/Debian:**
```bash
# Update package list
sudo apt-get update

# Install Docker
sudo apt-get install docker.io

# Start Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Verify installation
docker --version
```

**CentOS/RHEL:**
```bash
# Install Docker
sudo yum install docker

# Start Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Verify installation
docker --version
```

### Pull and run ONLYOFFICE Docs

```bash
# Pull the latest ONLYOFFICE Docs image
sudo docker pull onlyoffice/documentserver

# Run ONLYOFFICE Docs container
sudo docker run -i -t -d -p 80:80 \
    --name onlyoffice-docs \
    -v /app/onlyoffice/DocumentServer/logs:/var/log/onlyoffice \
    -v /app/onlyoffice/DocumentServer/data:/var/www/onlyoffice/Data \
    -v /app/onlyoffice/DocumentServer/lib:/var/lib/onlyoffice \
    onlyoffice/documentserver
```

### Verify installation

```bash
# Check if container is running
sudo docker ps

# Access ONLYOFFICE Docs
# Open browser and visit: http://your-server-ip
```

### Configure plugins directory

```bash
# Access the container
sudo docker exec -it onlyoffice-docs /bin/bash

# Navigate to plugins directory
cd /var/www/onlyoffice/documentserver/sdkjs-plugins

# Exit container
exit
```

### Mount plugins directory (recommended)

Stop and remove the existing container:
```bash
sudo docker stop onlyoffice-docs
sudo docker rm onlyoffice-docs
```

Run with plugins volume mounted:
```bash
sudo docker run -i -t -d -p 80:80 \
    --name onlyoffice-docs \
    -v /app/onlyoffice/DocumentServer/logs:/var/log/onlyoffice \
    -v /app/onlyoffice/DocumentServer/data:/var/www/onlyoffice/Data \
    -v /app/onlyoffice/DocumentServer/lib:/var/lib/onlyoffice \
    -v /app/onlyoffice/plugins:/var/www/onlyoffice/documentserver/sdkjs-plugins \
    onlyoffice/documentserver
```

Now place your plugins in `/app/onlyoffice/plugins/` on your host machine.

## Method 2: Package installation

### Ubuntu/Debian installation

```bash
# Add ONLYOFFICE repository GPG key
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys CB2DE8E5

# Add ONLYOFFICE repository
echo "deb https://download.onlyoffice.com/repo/debian squeeze main" | sudo tee /etc/apt/sources.list.d/onlyoffice.list

# Update package list
sudo apt-get update

# Install ONLYOFFICE Docs
sudo apt-get install onlyoffice-documentserver

# Verify installation
sudo systemctl status onlyoffice-documentserver
```

### CentOS/RHEL installation

```bash
# Add ONLYOFFICE repository
sudo yum install https://download.onlyoffice.com/repo/centos/main/noarch/onlyoffice-repo.noarch.rpm

# Install ONLYOFFICE Docs
sudo yum install onlyoffice-documentserver

# Verify installation
sudo systemctl status onlyoffice-documentserver
```

### Plugins directory location

```bash
# Plugins directory for package installation
cd /var/www/onlyoffice/documentserver/sdkjs-plugins

# Set proper permissions
sudo chown -R ds:ds /var/www/onlyoffice/documentserver/sdkjs-plugins
sudo chmod -R 755 /var/www/onlyoffice/documentserver/sdkjs-plugins
```

## Method 3: Docker Compose

For more complex setups with databases and services:

**docker-compose.yml:**
```yaml
version: '3'
services:
  onlyoffice-documentserver:
    image: onlyoffice/documentserver:latest
    container_name: onlyoffice-docs
    depends_on:
      - onlyoffice-postgresql
      - onlyoffice-rabbitmq
    environment:
      - DB_TYPE=postgres
      - DB_HOST=onlyoffice-postgresql
      - DB_PORT=5432
      - DB_NAME=onlyoffice
      - DB_USER=onlyoffice
      - AMQP_URI=amqp://guest:guest@onlyoffice-rabbitmq
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /app/onlyoffice/DocumentServer/logs:/var/log/onlyoffice
      - /app/onlyoffice/DocumentServer/data:/var/www/onlyoffice/Data
      - /app/onlyoffice/DocumentServer/lib:/var/lib/onlyoffice
      - /app/onlyoffice/plugins:/var/www/onlyoffice/documentserver/sdkjs-plugins
    stdin_open: true
    restart: always
    
  onlyoffice-postgresql:
    image: postgres:13
    container_name: onlyoffice-postgresql
    environment:
      - POSTGRES_DB=onlyoffice
      - POSTGRES_USER=onlyoffice
      - POSTGRES_PASSWORD=onlyoffice
    volumes:
      - postgresql_data:/var/lib/postgresql/data
    restart: always
    
  onlyoffice-rabbitmq:
    image: rabbitmq:3-management
    container_name: onlyoffice-rabbitmq
    restart: always

volumes:
  postgresql_data:
```

**Start services:**
```bash
sudo docker-compose up -d

# Check status
sudo docker-compose ps
```

## Configuring HTTPS (recommended)

### Using Let's Encrypt with Docker

```bash
# Install certbot
sudo apt-get install certbot

# Get SSL certificate
sudo certbot certonly --standalone -d your-domain.com

# Run Docker with SSL
sudo docker run -i -t -d -p 443:443 \
    --name onlyoffice-docs \
    -v /etc/letsencrypt/live/your-domain.com/fullchain.pem:/var/www/onlyoffice/Data/certs/onlyoffice.crt \
    -v /etc/letsencrypt/live/your-domain.com/privkey.pem:/var/www/onlyoffice/Data/certs/onlyoffice.key \
    -v /app/onlyoffice/plugins:/var/www/onlyoffice/documentserver/sdkjs-plugins \
    onlyoffice/documentserver
```

## Adding plugins to ONLYOFFICE Docs

### Copy plugin to server

```bash
# Create plugin folder
sudo mkdir -p /app/onlyoffice/plugins/my-plugin

# Copy plugin files (from your local machine)
scp -r ./my-plugin user@server-ip:/app/onlyoffice/plugins/

# Or if using Docker exec
sudo docker cp ./my-plugin onlyoffice-docs:/var/www/onlyoffice/documentserver/sdkjs-plugins/
```

### Set permissions

```bash
# For Docker
sudo docker exec onlyoffice-docs chown -R ds:ds /var/www/onlyoffice/documentserver/sdkjs-plugins/my-plugin
sudo docker exec onlyoffice-docs chmod -R 755 /var/www/onlyoffice/documentserver/sdkjs-plugins/my-plugin

# For package installation
sudo chown -R ds:ds /var/www/onlyoffice/documentserver/sdkjs-plugins/my-plugin
sudo chmod -R 755 /var/www/onlyoffice/documentserver/sdkjs-plugins/my-plugin
```

### Restart ONLYOFFICE Docs

```bash
# Docker
sudo docker restart onlyoffice-docs

# Package installation
sudo systemctl restart onlyoffice-documentserver
```

## Integrating with your application

### Basic integration example

```html
<!DOCTYPE html>
<html>
<head>
    <title>ONLYOFFICE Integration</title>
    <script src="http://your-server-ip/web-apps/apps/api/documents/api.js"></script>
</head>
<body>
    <div id="placeholder"></div>
    <script>
        new DocsAPI.DocEditor("placeholder", {
            "document": {
                "fileType": "docx",
                "key": "unique-document-key",
                "title": "Test Document.docx",
                "url": "http://your-server/document.docx"
            },
            "documentType": "word",
            "editorConfig": {
                "callbackUrl": "http://your-server/callback"
            }
        });
    </script>
</body>
</html>
```

## Testing plugins in ONLYOFFICE Docs

1. Add your plugin to the plugins directory
2. Restart the service
3. Open a document in the editor
4. Click the "Plugins" button
5. Find your plugin in the list
6. Test functionality

## Common installation issues

### Port already in use

**Error name:** Port 80 or 443 already in use

:::warning[Wrong]
```bash
# Running without checking port availability
sudo docker run -p 80:80 onlyoffice/documentserver
```
:::

:::tip[Correct]
```bash
# Check if port is available first
sudo lsof -i :80

# Use different port if needed
sudo docker run -p 8080:80 onlyoffice/documentserver

# Or stop service using the port
sudo systemctl stop apache2  # Example
```
:::

Error output: "docker: Error response from daemon: driver failed programming external connectivity on endpoint: Bind for 0.0.0.0:80 failed: port is already allocated."

### Insufficient disk space

**Error name:** Not enough disk space for installation

:::warning[Wrong]
```bash
# Installing without checking disk space
sudo apt-get install onlyoffice-documentserver
```
:::

:::tip[Correct]
```bash
# Check available disk space first
df -h

# Ensure at least 40 GB free
# Clean up if needed
sudo apt-get clean
sudo docker system prune

# Then install
sudo apt-get install onlyoffice-documentserver
```
:::

Error output: "No space left on device" during installation.

### Permission denied accessing plugins

**Error name:** Cannot write to plugins directory

:::warning[Wrong]
```bash
# Copying plugin without proper permissions
cp -r my-plugin /var/www/onlyoffice/documentserver/sdkjs-plugins/
```
:::

:::tip[Correct]
```bash
# Use sudo for system directories
sudo cp -r my-plugin /var/www/onlyoffice/documentserver/sdkjs-plugins/

# Set proper ownership
sudo chown -R ds:ds /var/www/onlyoffice/documentserver/sdkjs-plugins/my-plugin

# Set proper permissions
sudo chmod -R 755 /var/www/onlyoffice/documentserver/sdkjs-plugins/my-plugin
```
:::

Error output: "Permission denied" when copying files.

### Container fails to start

**Error name:** Docker container exits immediately

:::warning[Wrong]
```bash
# Running container without checking logs
sudo docker run onlyoffice/documentserver
# Container exits...
```
:::

:::tip[Correct]
```bash
# Check container logs
sudo docker logs onlyoffice-docs

# Common issues:
# 1. Port conflict - use different port
# 2. Insufficient memory - increase RAM
# 3. Missing volumes - check mount points

# Run with proper configuration
sudo docker run -i -t -d -p 80:80 \
    --memory="4g" \
    -v /app/onlyoffice/DocumentServer/logs:/var/log/onlyoffice \
    onlyoffice/documentserver
```
:::

Error output: Container status shows "Exited (1)" - check logs for specific error.

## Managing ONLYOFFICE Docs

### Start/stop/restart

**Docker:**
```bash
sudo docker stop onlyoffice-docs
sudo docker start onlyoffice-docs
sudo docker restart onlyoffice-docs
```

**Package installation:**
```bash
sudo systemctl stop onlyoffice-documentserver
sudo systemctl start onlyoffice-documentserver
sudo systemctl restart onlyoffice-documentserver
```

### View logs

**Docker:**
```bash
# Live logs
sudo docker logs -f onlyoffice-docs

# Last 100 lines
sudo docker logs --tail 100 onlyoffice-docs
```

**Package installation:**
```bash
# View logs
sudo tail -f /var/log/onlyoffice/documentserver/*.log
```

### Update ONLYOFFICE Docs

**Docker:**
```bash
# Pull latest image
sudo docker pull onlyoffice/documentserver:latest

# Stop and remove old container
sudo docker stop onlyoffice-docs
sudo docker rm onlyoffice-docs

# Run new version (with same volumes)
sudo docker run -i -t -d -p 80:80 \
    --name onlyoffice-docs \
    -v /app/onlyoffice/DocumentServer/logs:/var/log/onlyoffice \
    -v /app/onlyoffice/DocumentServer/data:/var/www/onlyoffice/Data \
    onlyoffice/documentserver:latest
```

**Package installation:**
```bash
# Update
sudo apt-get update
sudo apt-get upgrade onlyoffice-documentserver
```

## Performance optimization

### Increase number of workers

Edit `/etc/onlyoffice/documentserver/local.json`:
```json
{
  "services": {
    "CoAuthoring": {
      "server": {
        "workers": 4
      }
    }
  }
}
```

### Configure caching

```json
{
  "services": {
    "CoAuthoring": {
      "cache": {
        "type": "memcached",
        "host": "localhost"
      }
    }
  }
}
```

## Security best practices

1. **Use HTTPS** - Always use SSL/TLS in production
2. **Configure firewall** - Limit access to necessary ports
3. **Regular updates** - Keep ONLYOFFICE Docs up to date
4. **Secure credentials** - Use strong passwords for services
5. **Enable JWT** - Use JSON Web Tokens for API security

## Next steps

- Set up [test environment](./test-environment-setup.md)
- Learn about [Cloud/SaaS installation](./cloud-saas-installation.md)
- Configure your [development workflow](/docs/plugin-and-macros/development-workflow/overview.md)

## Additional resources

- **Official documentation**: [https://helpcenter.onlyoffice.com/installation/docs-developer-install.aspx](https://helpcenter.onlyoffice.com/installation/docs-developer-install.aspx)
- **Docker Hub**: [https://hub.docker.com/r/onlyoffice/documentserver](https://hub.docker.com/r/onlyoffice/documentserver)
- **API documentation**: [https://api.onlyoffice.com](https://api.onlyoffice.com)
- **GitHub**: [https://github.com/ONLYOFFICE/DocumentServer](https://github.com/ONLYOFFICE/DocumentServer)

## Conclusion

ONLYOFFICE Docs on-premises installation provides a robust environment for testing plugins in production-like conditions. With full control over the server and configuration, you can thoroughly test and optimize your plugins before deployment.
