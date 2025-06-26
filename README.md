# Unregistry GitHub Action

Push Docker images directly to your servers over SSH without needing a registry.

This action wraps [unregistry](https://github.com/psviderski/unregistry) to transfer Docker images from GitHub runners straight to remote Docker hosts. Ideal for deploying to staging and production servers after building images in CI.

## Quick Start

```yaml
name: Deploy
on: [push]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build image
        run: docker build -t myapp:${{ github.sha }} .

      - name: Add server to known hosts
        run: ssh-keyscan -H prod-server.com >> ~/.ssh/known_hosts

      - name: Deploy to server
        uses: sonofbytes/unregistry-action@v0.1.0
        with:
          image: myapp:${{ github.sha }}
          destination: deploy@prod-server.com
          ssh_key: ${{ secrets.SSH_PRIVATE_KEY }}
```

### Using SSH Password Authentication

```yaml
name: Deploy
on: [push]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build image
        run: docker build -t myapp:${{ github.sha }} .

      - name: Add server to known hosts
        run: ssh-keyscan -H prod-server.com >> ~/.ssh/known_hosts

      - name: Deploy to server
        uses: sonofbytes/unregistry-action@v0.1.0
        with:
          image: myapp:${{ github.sha }}
          destination: deploy@prod-server.com
          ssh_password: ${{ secrets.SSH_PASSWORD }}
```

## Why Use This

- **Skip registry complexity** - No need for Docker Hub, ECR, or private registries
- **Direct deployment** - Images go straight from CI to your servers
- **Efficient transfers** - Only missing layers are sent
- **Standard SSH** - Uses existing SSH infrastructure and keys

## Inputs

| Name           | Description                                         | Required | Default |
| -------------- | --------------------------------------------------- | -------- | ------- |
| `image`        | Docker image name:tag (must exist locally)          | Yes      | -       |
| `destination`  | SSH destination (user@host or user@host:port)       | Yes      | -       |
| `ssh_key`      | SSH private key content                             | No       | -       |
| `ssh_password` | SSH password for authentication                     | No       | -       |
| `platform`     | Platform for multi-arch images (e.g. linux/amd64)  | No       | -       |

**Note**: You cannot specify both `ssh_key` and `ssh_password` at the same time. Choose one authentication method.

## Examples

### Basic deployment with SSH key

```yaml
- name: Deploy to production
  uses: sonofbytes/unregistry-action@v0.1.0
  with:
    image: webapp:latest
    destination: root@server.example.com
    # destination: deploy@10.0.0.100:2222  # for custom SSH port
    ssh_key: ${{ secrets.SERVER_SSH_KEY }}
    # platform: linux/arm64  # for multi-platform images
```

### Basic deployment with SSH password

```yaml
- name: Deploy to production
  uses: sonofbytes/unregistry-action@v0.1.0
  with:
    image: webapp:latest
    destination: root@server.example.com
    ssh_password: ${{ secrets.SERVER_SSH_PASSWORD }}
    # platform: linux/arm64  # for multi-platform images
```

### Using SSH agent

```yaml
- name: Setup SSH
  uses: webfactory/ssh-agent@v0.7.0
  with:
    ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

- name: Deploy
  uses: sonofbytes/unregistry-action@v0.1.0
  with:
    image: myapp:latest
    destination: deploy@server.com
    # ssh_key not needed when using SSH agent
```

## Requirements

**GitHub runner:**

- Linux (ubuntu-latest recommended)
- Docker CLI 19.03+
- Network access to target servers

**Remote server:**

- Docker daemon running
- SSH server accepting connections
- SSH user can run Docker commands (root user or in docker group)

## SSH Setup

### Option 1: SSH Key Authentication (Recommended)

Generate an SSH key pair if needed:

```bash
ssh-keygen -t rsa -b 4096 -C "github-actions"
```

Add the public key to your server:

```bash
ssh-copy-id user@server.com
```

Add the private key to GitHub repository secrets (Settings → Secrets → Actions).

### Option 2: SSH Password Authentication

For password authentication, ensure your server allows password authentication:

1. Edit `/etc/ssh/sshd_config` on your server:
   ```
   PasswordAuthentication yes
   ```

2. Restart SSH service:
   ```bash
   sudo systemctl restart ssh
   ```

3. Add the SSH password to GitHub repository secrets (Settings → Secrets → Actions).

**Security Note**: SSH key authentication is generally more secure than password authentication and is the recommended approach.

## Avoiding Host Key Errors

Add your server's host key to avoid verification failures:

```yaml
- name: Add server to known hosts
  run: ssh-keyscan -H your-server.com >> ~/.ssh/known_hosts
```

## Troubleshooting

**Permission denied (SSH key):** Check that your SSH key is correct and the public key is installed on the server.

**Permission denied (SSH password):** Verify the password is correct and that PasswordAuthentication is enabled in the server's SSH configuration.

**Cannot connect to Docker daemon:** Ensure Docker is running and the SSH user can execute Docker commands.

**Image not found:** Build or pull the image before running this action.

**Connection timeout:** Verify network connectivity and SSH port accessibility.

**sshpass not found:** This error should not occur as the action automatically installs sshpass when using password authentication.

**Both ssh_key and ssh_password provided:** The action will fail with an error. Use only one authentication method.

## How It Works

1. Establishes SSH tunnel to your server
2. Starts temporary unregistry container on the remote server
3. Forwards local port to the unregistry container over SSH
4. Runs `docker push` to the forwarded port, transferring only missing layers
5. Stops the unregistry container and closes the tunnel

The unregistry container acts as a temporary registry interface for your remote Docker daemon, so `docker push` works normally while transferring images directly over SSH.

## Contributing

Issues and pull requests welcome. This action wraps [unregistry](https://github.com/psviderski/unregistry) by Pasha Sviderski.

## License

Apache License 2.0
