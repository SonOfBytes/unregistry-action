# Unregistry (docker-pussh) GitHub Action

> **Push Docker images directly to remote servers over SSH** - Skip the registry complexity and deploy containers directly to your servers.

This GitHub Action wraps the powerful [unregistry](https://github.com/psviderski/unregistry) tool, allowing you to push Docker images from your CI/CD pipeline straight to remote Docker hosts without needing a registry. Perfect for deploying to staging and production servers instantly after building your images.

## ✨ Features

- **🚀 Direct Transfer**: Push images from GitHub runners to remote Docker hosts over SSH
- **⚡ Efficient**: Only missing layers are transferred, making deployments fast
- **🔒 Secure**: Uses SSH for encrypted connections with support for SSH keys
- **🌐 Multi-Platform**: Supports pushing specific architectures from multi-arch images
- **🔧 Zero Registry**: No need for Docker Hub, ECR, or any registry service
- **📦 Lightweight**: Minimal overhead with no container startup delays

## 🚀 Quick Start

```yaml
name: Build and Deploy
on: [push]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .
      
      - name: Push to production server
        uses: sonofbytes/unregistry-action@v0.1.0
        with:
          image: myapp:${{ github.sha }}
          destination: deploy@prod-server.com
          ssh_key: ${{ secrets.SSH_PRIVATE_KEY }}
```

## 📋 Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `image` | Docker image name:tag to push (must exist locally) | ✅ Yes | - |
| `destination` | SSH destination (user@host or user@host:port) | ✅ Yes | - |
| `ssh_key` | SSH private key for authentication | ❌ No | - |
| `platform` | Specific platform for multi-arch images (e.g., `linux/amd64`) | ❌ No | - |

### Input Details

**`image`**: The Docker image that must already exist in the runner's Docker daemon. Build or pull the image in a previous step.

**`destination`**: SSH connection details in the format:
- `user@hostname` (uses default port 22)
- `user@hostname:2222` (custom SSH port)
- `hostname` (uses current user)

**`ssh_key`**: Private SSH key content (usually from GitHub Secrets). If omitted, assumes SSH access is already configured (SSH agent, known hosts, etc.).

**`platform`**: For multi-architecture images, specify which platform to push (e.g., `linux/amd64`, `linux/arm64`).

## 📖 Usage Examples

### Basic Deployment

```yaml
- name: Deploy to server
  uses: sonofbytes/unregistry-action@v0.1.0
  with:
    image: myapp:latest
    destination: root@server.example.com
    ssh_key: ${{ secrets.SERVER_SSH_KEY }}
```

### Multi-Platform Deployment

```yaml
- name: Deploy ARM64 image
  uses: sonofbytes/unregistry-action@v0.1.0
  with:
    image: myapp:latest
    destination: ubuntu@arm-server.com
    platform: linux/arm64
    ssh_key: ${{ secrets.ARM_SERVER_KEY }}
```

### Custom SSH Port

```yaml
- name: Deploy to server with custom port
  uses: sonofbytes/unregistry-action@v0.1.0
  with:
    image: webapp:${{ github.sha }}
    destination: deploy@10.0.0.100:2222
    ssh_key: ${{ secrets.DEPLOY_KEY }}
```

### Using SSH Agent (No Key Required)

```yaml
- name: Setup SSH Agent
  uses: webfactory/ssh-agent@v0.7.0
  with:
    ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

- name: Deploy without explicit key
  uses: sonofbytes/unregistry-action@v0.1.0
  with:
    image: myapp:latest
    destination: deploy@server.com
    # No ssh_key needed - using SSH agent
```

### Complete CI/CD Pipeline

```yaml
name: Build and Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Build image
        run: |
          docker build -t myapp:${{ github.sha }} .
          docker tag myapp:${{ github.sha }} myapp:latest
      
      - name: Add SSH host key
        run: |
          ssh-keyscan -H prod-server.com >> ~/.ssh/known_hosts
      
      - name: Deploy to production
        uses: sonofbytes/unregistry-action@v0.1.0
        with:
          image: myapp:latest
          destination: deploy@prod-server.com
          ssh_key: ${{ secrets.PROD_SSH_KEY }}
      
      - name: Deploy to staging
        uses: sonofbytes/unregistry-action@v0.1.0
        with:
          image: myapp:${{ github.sha }}
          destination: deploy@staging-server.com
          ssh_key: ${{ secrets.STAGING_SSH_KEY }}
```

## ⚙️ Requirements

### GitHub Runner Requirements

- **OS**: Linux (`ubuntu-latest`) runners
- **Docker**: Docker CLI 19.03+ (pre-installed on GitHub runners)
- **SSH**: OpenSSH client (pre-installed on GitHub runners)
- **Network**: Outbound access to target servers

> ⚠️ **Windows and macOS runners are not supported**. Use `ubuntu-latest` for reliable Docker and SSH support.

### Remote Server Requirements

- **Docker**: Docker daemon running and accessible
- **SSH**: SSH server accepting connections
- **Permissions**: SSH user must be able to run Docker commands

The SSH user should be able to run `docker ps` without password prompts. This typically means:
- Using the `root` user, or
- Adding the user to the `docker` group: `sudo usermod -aG docker username`
- Configuring passwordless sudo for Docker commands if needed

## 🔐 Security Setup

### SSH Key Management

1. **Generate SSH Key Pair** (if you don't have one):
   ```bash
   ssh-keygen -t rsa -b 4096 -C "github-actions@yourproject.com"
   ```

2. **Add Public Key to Server**:
   ```bash
   ssh-copy-id user@server.com
   # or manually add to ~/.ssh/authorized_keys
   ```

3. **Add Private Key to GitHub Secrets**:
   - Go to your repository → Settings → Secrets and variables → Actions
   - Click "New repository secret"
   - Name: `SSH_PRIVATE_KEY` (or similar)
   - Value: The entire private key content (including headers)

### SSH Host Key Verification

To avoid "Host key verification failed" errors, add the server's host key:

```yaml
- name: Add SSH host key
  run: ssh-keyscan -H your-server.com >> ~/.ssh/known_hosts
```

Or disable strict checking (less secure):
```yaml
- name: Configure SSH
  run: |
    mkdir -p ~/.ssh
    echo "StrictHostKeyChecking no" >> ~/.ssh/config
```

## 🛠️ Troubleshooting

### Common Issues

**❌ Permission denied (publickey)**
- Verify SSH key is correct and properly added to GitHub Secrets
- Ensure public key is installed on the target server
- Check that the SSH user exists and has proper permissions

**❌ Cannot connect to Docker daemon**
- Ensure Docker is running on the remote server
- Verify the SSH user can run Docker commands
- Add user to `docker` group or configure passwordless sudo

**❌ Host key verification failed**
- Add the server's host key to known_hosts (see Security Setup)
- Or configure SSH to skip host key checking

**❌ Image not found**
- Ensure the Docker image exists locally before running this action
- Check that the image name and tag are correct
- Verify the image was built successfully in previous steps

**❌ Connection timeout**
- Check network connectivity between GitHub runners and your server
- Verify the SSH port is correct and accessible
- Ensure firewall rules allow SSH connections

### Debug Mode

Enable debug logging by setting the `ACTIONS_STEP_DEBUG` secret to `true` in your repository settings.

### Platform-Specific Issues

**Multi-arch images not working:**
- Ensure your Docker setup supports multi-platform images
- Use Docker Buildx for building multi-arch images
- Verify the target platform exists in your image manifest

## 📚 How It Works

This action uses [unregistry](https://github.com/psviderski/unregistry) under the hood, which:

1. **Establishes SSH Connection**: Connects to your remote server securely
2. **Analyzes Local Image**: Examines layers in your local Docker image
3. **Transfers Missing Layers**: Only sends layers that don't exist remotely (efficient!)
4. **Reconstructs Image**: Assembles the complete image on the remote Docker daemon

The process is similar to `docker push` but uses SSH instead of a registry, making it perfect for direct server deployments.

## 🔄 Versioning

This action's versions correspond to [unregistry releases](https://github.com/psviderski/unregistry/releases):

- `@v0.1.0` - Uses unregistry v0.1.0
- `@main` - Latest development version (not recommended for production)

We recommend pinning to a specific version tag for stability.

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guidelines](CONTRIBUTING.md) for details.

## 📄 License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.

## 🙏 Credits

This action is a wrapper around [unregistry](https://github.com/psviderski/unregistry) by [Pasha Sviderski](https://github.com/psviderski). 

If you find this tool helpful:
- ⭐ Star this repository
- ⭐ Star the [upstream unregistry project](https://github.com/psviderski/unregistry)
- 🐛 Report issues or suggest improvements

---

**Happy container deploying! 🚀**

*Skip the registry dance and push directly to your servers with confidence.*