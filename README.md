# GitLab CI/CD Local Development POC

This project demonstrates two approaches for running GitLab CI/CD pipelines locally on Windows 11:

1. **gitlab-ci-local** (Recommended for local development) - Simulates GitLab CI/CD without requiring a GitLab Runner service
2. **GitLab Runner in Docker** (Educational/Infrastructure learning) - Sets up actual GitLab Runner infrastructure

## Table of Contents
- [Quick Start (Recommended)](#quick-start-recommended)
- [Prerequisites](#prerequisites)
- [Method 1: gitlab-ci-local (Recommended)](#method-1-gitlab-ci-local-recommended)
- [Method 2: GitLab Runner Setup (Educational)](#method-2-gitlab-runner-setup-educational)
- [Example Usage](#example-usage)
- [Troubleshooting](#troubleshooting)
- [Resources](#resources)

## Quick Start (Recommended)

For most local development needs, use `gitlab-ci-local`:

```cmd
# Install Node.js from nodejs.org, then:
npm install -g gitlab-ci-local

# In your project directory with .gitlab-ci.yml:
gitlab-ci-local
```

**That's it!** No GitLab Runner container needed for local development.

## Prerequisites

### For gitlab-ci-local (Recommended)
- **Node.js** - Download from [nodejs.org](https://nodejs.org/)
- **Docker Desktop** - Download from [Docker Desktop](https://www.docker.com/products/docker-desktop/)

### For GitLab Runner Setup (Educational Only)
- **Docker Desktop for Windows** - Ensure it's running with Linux containers
- **WSL 2** - Run `wsl --install` in PowerShell as Administrator
- **Git** (Optional) - Download from [Git for Windows](https://git-scm.com/download/win)

## Method 1: gitlab-ci-local (Recommended)

**Use this method for actual local development and testing.**

### Installation
```cmd
# Install Node.js from nodejs.org first, then:
npm install -g gitlab-ci-local
```

### Usage
```cmd
# In your project directory with .gitlab-ci.yml
gitlab-ci-local

# Run specific job
gitlab-ci-local --job test

# List all available jobs
gitlab-ci-local --list

# Run with variables
gitlab-ci-local --variable NODE_VERSION=18
```

### Why use gitlab-ci-local?
- ✅ **Simple setup** - No GitLab Runner service required
- ✅ **Fast execution** - Direct Docker integration
- ✅ **Perfect for development** - Test your `.gitlab-ci.yml` locally
- ✅ **Windows-friendly** - Designed for local development

## Method 2: GitLab Runner Setup (Educational)

**Use this method only for learning about GitLab CI/CD infrastructure or testing actual runner registration.**

⚠️ **Note**: This method does NOT enable local pipeline execution. It's for educational purposes and understanding GitLab Runner infrastructure.

### GitLab Runner Container Setup

1. **Verify Docker Installation**
```cmd
docker --version
docker run hello-world
```

2. **Pull GitLab Runner Docker Image**
```cmd
docker pull gitlab/gitlab-runner:latest
```

3. **Create GitLab Runner Container**
```cmd
docker run -d --name gitlab-runner --restart always ^
  -v /var/run/docker.sock:/var/run/docker.sock ^
  -v gitlab-runner-config:/etc/gitlab-runner ^
  gitlab/gitlab-runner:latest
```

4. **Create GitLab Runner Configuration**
```cmd
docker exec gitlab-runner bash -c "mkdir -p /etc/gitlab-runner && cat > /etc/gitlab-runner/config.toml << EOF
concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800
EOF"
```

5. **Restart and Verify**
```cmd
docker restart gitlab-runner
docker ps
```

### What This Achieves
- ✅ **Learning experience** - Understand GitLab Runner infrastructure
- ✅ **Runner registration testing** - Practice registering with GitLab instances
- ✅ **CI/CD concepts** - Learn how GitLab CI/CD works behind the scenes
- ❌ **Local pipeline execution** - This does NOT enable local `.gitlab-ci.yml` testing

## Example Usage

### Sample .gitlab-ci.yml
Create a `.gitlab-ci.yml` file in your project root:

```yaml
stages:
  - test
  - build

variables:
  NODE_VERSION: "18"

test:
  stage: test
  image: node:18
  script:
    - echo "Running tests..."
    - npm --version
    - node --version
    - echo "Tests completed successfully!"

build:
  stage: build
  image: node:18
  script:
    - echo "Building application..."
    - npm --version
    - echo "Build completed successfully!"
  artifacts:
    paths:
      - dist/
    expire_in: 1 hour
```

### Running the Example

**Using gitlab-ci-local (Recommended):**
```cmd
# Install gitlab-ci-local (requires Node.js)
npm install -g gitlab-ci-local

# Run all jobs
gitlab-ci-local

# Run specific job
gitlab-ci-local --job test
```

**Using GitLab Runner container (Educational only):**
The GitLab Runner container can be used to understand runner concepts, but cannot execute local pipelines. To register it with an actual GitLab instance:

```cmd
# Register with GitLab instance (requires GitLab URL and registration token)
docker exec -it gitlab-runner gitlab-runner register
```

## Troubleshooting

### Common Issues

**1. Docker not running**
- Ensure Docker Desktop is started and running
- Check system tray for Docker icon

**2. Permission denied errors**
- Run PowerShell/Command Prompt as Administrator
- Ensure Docker has proper permissions

**3. Container not found**
```cmd
# Check if container exists
docker ps -a

# Recreate container if needed
docker rm gitlab-runner
# Then run the create command again
```

**4. WSL 2 issues**
```powershell
# Update WSL
wsl --update

# Set default version
wsl --set-default-version 2
```

**5. Mount path issues on Windows**
- Use full paths with forward slashes
- For Command Prompt: `%cd%` for current directory
- For PowerShell: `${PWD}` for current directory

**6. Config file errors (ERROR: Failed to load config stat /etc/gitlab-runner/config.toml)**
This error occurs when GitLab Runner starts without a configuration file. The solution is to create a basic config file:

```cmd
# Create the configuration file
docker exec gitlab-runner bash -c "mkdir -p /etc/gitlab-runner && cat > /etc/gitlab-runner/config.toml << EOF
concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800
EOF"

# Restart the container
docker restart gitlab-runner
```

**Why this works:**
- GitLab Runner requires a valid `config.toml` file to start properly
- The `concurrent = 1` setting allows one job to run at a time
- The `check_interval = 0` disables automatic job polling (useful for local execution)
- The `session_server` section enables session management for debugging
- Even without registered runners, this minimal config allows GitLab Runner to start and accept local execution commands

**7. gitlab-ci-local rsync error on Windows**
When using `gitlab-ci-local` on Windows, you may encounter an rsync error when copying artifacts:

```
Error: Command failed with exit code 1: rsync --exclude=/.gitlab-ci-reports/ -a ...
'rsync' is not recognized as an internal or external command
```

**Solutions:**

Option 1: Install rsync via WSL/Git Bash
```cmd
# If using Git Bash, rsync should be available
# Run gitlab-ci-local from Git Bash instead of PowerShell
```

Option 2: Use WSL
```cmd
# Run from WSL where rsync is available
wsl
cd /mnt/h/code/yl/gitlab-runner-poc
gitlab-ci-local
```

Option 3: Disable artifacts (if not needed)
```yaml
# In .gitlab-ci.yml, remove or comment out artifacts section
build:
  stage: build
  image: node:18
  script:
    - echo "Building application..."
    - npm --version
    - echo "Build completed successfully!"
  # artifacts:
  #   paths:
  #     - dist/
  #   expire_in: 1 hour
```

**Note:** The jobs still run successfully even with the rsync error. This only affects artifact copying, not the actual pipeline execution.

### Logs and Debugging

View GitLab Runner logs:
```cmd
docker logs gitlab-runner
```

Execute commands interactively:
```cmd
docker exec -it gitlab-runner bash
```

## Useful Commands

### Container Management
```cmd
# Start GitLab Runner
docker start gitlab-runner

# Stop GitLab Runner
docker stop gitlab-runner

# Remove GitLab Runner container
docker rm gitlab-runner

# View container logs
docker logs gitlab-runner -f
```

### Pipeline Testing with gitlab-ci-local
```cmd
# List all jobs in .gitlab-ci.yml
gitlab-ci-local --list

# Run with specific variables
gitlab-ci-local --variable NODE_VERSION=18 --job test

# Run with environment variables
gitlab-ci-local --env VAR_NAME=value --job test

# Show what would run without executing
gitlab-ci-local --preview
```

## Resources

- [GitLab Runner Documentation](https://docs.gitlab.com/runner/)
- [GitLab CI/CD Documentation](https://docs.gitlab.com/ee/ci/)
- [Docker Desktop for Windows](https://docs.docker.com/desktop/windows/)
- [gitlab-ci-local Tool](https://github.com/firecow/gitlab-ci-local)

## Contributing

Feel free to submit issues and enhancement requests for this POC project!

## License

This project is for educational and testing purposes.