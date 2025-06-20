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

## Why Use This

- **Skip registry complexity** - No need for Docker Hub, ECR, or private registries
- **Direct deployment** - Images go straight from CI to your servers
- **Efficient transfers** - Only missing layers are sent
- **Standard SSH** - Uses existing SSH infrastructure and keys

## Inputs

| Name          | Description                                       | Required | Default |
| ------------- | ------------------------------------------------- | -------- | ------- |
| `image`       | Docker image name:tag (must exist locally)        | Yes      | -       |
| `destination` | SSH destination (user@host or user@host:port)     | Yes      | -       |
| `ssh_key`     | SSH private key content                           | No       | -       |
| `platform`    | Platform for multi-arch images (e.g. linux/amd64) | No       | -       |

## Examples

### Basic deployment

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

Generate an SSH key pair if needed:

```bash
ssh-keygen -t rsa -b 4096 -C "github-actions"
```

Add the public key to your server:

```bash
ssh-copy-id user@server.com
```

Add the private key to GitHub repository secrets (Settings → Secrets → Actions).

## Avoiding Host Key Errors

Add your server's host key to avoid verification failures:

```yaml
- name: Add server to known hosts
  run: ssh-keyscan -H your-server.com >> ~/.ssh/known_hosts
```

## Troubleshooting

**Permission denied:** Check that your SSH key is correct and the public key is installed on the server.

**Cannot connect to Docker daemon:** Ensure Docker is running and the SSH user can execute Docker commands.

**Image not found:** Build or pull the image before running this action.

**Connection timeout:** Verify network connectivity and SSH port accessibility.

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
