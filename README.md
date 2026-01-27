

# BoostFlow Deployment

A comprehensive deployment solution for BoostFlow using Ansible and Docker.

## Project Structure

```
ansible-boostflow/
├── README.md                    # This file
├── DEPLOYMENT_NOTES.md          # Deployment configuration notes
├── .env.local                   # Environment variables (not in repo)
├── .gitignore                   # Git ignore rules
├── .dockerignore               # Docker ignore rules
├── ansible/                     # Ansible configuration
│   ├── README.md               # Ansible documentation
│   ├── ansible.cfg             # Ansible configuration
│   ├── inventories/            # Inventory files
│   │   ├── hosts.yaml          # Host definitions
│   │   ├── group_vars/         # Group variables
│   │   │   ├── all.yaml        # Global variables
│   │   │   └── azure_hosts.yaml # Azure-specific variables
│   │   └── host_vars/          # Host-specific variables
│   │       └── devops-vm.yaml  # VM-specific variables
│   ├── playbooks/              # Ansible playbooks
│   │   ├── site.yaml           # Main playbook
│   │   ├── deploy_docker.yaml  # Docker deployment
│   │   ├── docker.yaml         # Docker installation
│   │   ├── install_kubernetes.yaml # Kubernetes setup
│   │   ├── install_nginx.yaml  # Nginx installation
│   │   ├── nextjs.yaml         # Next.js deployment
│   │   └── nodejs.yaml         # Node.js setup
│   └── templates/              # Jinja2 templates
│       ├── nextjs.service.j2   # Systemd service template
│       ├── nginx.https.j2      # HTTPS Nginx config
│       ├── nginx.nextjs.j2     # Next.js Nginx config
│       └── nginx.react.j2      # React Nginx config
└── docker/                     # Docker configuration
    ├── README.md               # Docker documentation
    ├── Dockerfile              # Multi-stage build
    ├── docker-compose.yaml     # Docker Compose config
    ├── nginx/                  # Nginx configuration
    │   ├── nginx.conf          # Main Nginx config
    │   └── default.conf        # Server block config
    └── scripts/                # Container scripts
        └── start.sh            # Startup script
```

## Quick Start

### Prerequisites

1. **Ansible** installed on your local machine
2. **Docker** and **Docker Compose** on target servers
3. **SSH access** to target servers
4. **SSH key** for GitHub access
5. **`.env.local`** file with your environment variables

### Environment Setup

1. Clone this repository:
   ```bash
   git clone <repository-url>
   cd ansible-boostflow
   ```

2. Create your `.env.local` file:
   ```bash
   cp .env.example .env.local
   # Edit .env.local with your actual values
   ```

3. Configure your inventory:
   ```bash
   # Edit ansible/inventories/hosts.yaml
   # Update IP addresses, SSH keys, etc.
   ```

### Deployment Options

#### Option 1: Docker Deployment (Recommended)

Deploys BoostFlow using Docker containers with combined Nginx + Next.js container and separate MinIO container.

```bash
cd ansible
ansible-playbook -i inventories/hosts.yaml playbooks/deploy_docker.yaml
```

**Features:**
- Combined Nginx + Next.js container for efficiency
- Separate MinIO container for object storage
- Health checks and monitoring
- Automatic SSL support (when configured)
- Easy scaling and management

#### Option 2: Traditional Deployment

Installs services directly on the host system.

```bash
cd ansible
ansible-playbook -i inventories/hosts.yaml playbooks/site.yaml
```

**Features:**
- Direct installation on host
- Systemd service management
- Traditional Nginx configuration
- Full system integration

#### Option 3: Kubernetes Deployment

Sets up Kubernetes cluster and deploys BoostFlow.

```bash
cd ansible
ansible-playbook -i inventories/hosts.yaml playbooks/install_kubernetes.yaml
# Then use the kubernetes-boostflow project for deployment
```

## Configuration

### Global Variables

Edit `ansible/inventories/group_vars/all.yaml`:

```yaml
app_name: "Boostflow"
app_repo: "git@github.com:Kdim67/BoostFlow.git"
app_port: 3000
branch: "main"
ssl_enabled: false  # Set to true for HTTPS
```

### Host-Specific Variables

Edit `ansible/inventories/host_vars/devops-vm.yaml`:

```yaml
ssl_enabled: true  # Override global setting
```

### Environment Variables

Create `.env.local` with your configuration:

```env
# Database
DATABASE_URL=your_database_url

# Authentication
NEXTAUTH_SECRET=your_secret
NEXTAUTH_URL=http://localhost:3000

# MinIO
MINIO_ACCESS_KEY=your_access_key
MINIO_SECRET_KEY=your_secret_key

# Other services
GOOGLE_MAPS_API_KEY=your_api_key
```

## Docker Architecture

The new Docker setup uses a two-container architecture:

### App Container (boostflow-app)
- **Base**: nginx:alpine with Node.js installed
- **Services**: Nginx (port 80/443) + Next.js (port 3000)
- **Features**: Reverse proxy, static file serving, health checks
- **Startup**: Custom script manages both services

### MinIO Container (boostflow-minio)
- **Base**: minio/minio:latest
- **Services**: MinIO server (port 9000) + Console (port 9001)
- **Features**: S3-compatible storage, web console
- **Storage**: Persistent Docker volume

## Monitoring and Maintenance

### Health Checks

```bash
# Check application health
curl http://your-server/health

# Check MinIO health
curl http://your-server:9000/minio/health/live

# View container status
docker-compose -f docker/docker-compose.yaml ps
```

### Logs

```bash
# View all logs
docker-compose -f docker/docker-compose.yaml logs -f

# View specific service logs
docker-compose -f docker/docker-compose.yaml logs -f app
docker-compose -f docker/docker-compose.yaml logs -f minio
```

### Updates

```bash
# Update application
cd ansible
ansible-playbook -i inventories/hosts.yaml playbooks/deploy_docker.yaml

# Or manually
docker-compose -f docker/docker-compose.yaml pull
docker-compose -f docker/docker-compose.yaml up -d
```

## Security Considerations

1. **Environment Variables**: Never commit `.env.local` to version control
2. **SSH Keys**: Use dedicated SSH keys for deployment
3. **SSL/TLS**: Enable SSL for production deployments
4. **Firewall**: Configure appropriate firewall rules
5. **Updates**: Keep Docker images and system packages updated

## Troubleshooting

### Common Issues

1. **Port Conflicts**: Ensure ports 80, 443, 9000, 9001 are available
2. **Permission Issues**: Check file permissions and ownership
3. **Network Issues**: Verify Docker network configuration
4. **SSL Issues**: Check certificate validity and configuration

### Debug Commands

```bash
# Check Ansible connectivity
ansible -i ansible/inventories/hosts.yaml all -m ping

# Test Docker Compose configuration
docker-compose -f docker/docker-compose.yaml config

# Access container shell
docker exec -it boostflow-app sh

# View Nginx configuration
docker exec boostflow-app nginx -t
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## Support

For issues and questions:
1. Check the troubleshooting section
2. Review logs for error messages
3. Open an issue on GitHub
4. Contact the development team
