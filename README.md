# GitLab Runner POC

This project demonstrates how to run GitLab CI/CD pipelines locally using GitLab Runner in Docker on Windows 11.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Setup Instructions](#setup-instructions)
- [Running GitLab Runner](#running-gitlab-runner)
- [Local Pipeline Execution](#local-pipeline-execution)
- [Example Usage](#example-usage)
- [Troubleshooting](#troubleshooting)
- [Resources](#resources)

## Prerequisites

Before starting, ensure you have the following installed on your Windows 11 machine:

### 1. Docker Desktop for Windows
- Download from [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- Install and ensure it's running
- Make sure Docker is using Linux containers (not Windows containers)

### 2. WSL 2 (Windows Subsystem for Linux)
Open PowerShell as Administrator and run:
```powershell
wsl --install
```
Restart your computer if prompted.

### 3. Git (Optional but recommended)
- Download from [Git for Windows](https://git-scm.com/download/win)

## Setup Instructions

### Step 1: Verify Docker Installation
Open PowerShell or Command Prompt and verify Docker is working:
```cmd
docker --version
docker run hello-world
```

### Step 2: Pull GitLab Runner Docker Image
```cmd
docker pull gitlab/gitlab-runner:latest
```

### Step 3: Create GitLab Runner Container
Create a persistent GitLab Runner container:

```cmd
docker run -d --name gitlab-runner --restart always ^
  -v /var/run/docker.sock:/var/run/docker.sock ^
  -v gitlab-runner-config:/etc/gitlab-runner ^
  gitlab/gitlab-runner:latest
```

**For PowerShell, use:**
```powershell
docker run -d --name gitlab-runner --restart always `
  -v /var/run/docker.sock:/var/run/docker.sock `
  -v gitlab-runner-config:/etc/gitlab-runner `
  gitlab/gitlab-runner:latest
```

### Step 4: Verify Installation
Check that the container is running:
```cmd
docker ps
```

You should see the `gitlab-runner` container in the list.

## Running GitLab Runner

### Check GitLab Runner Version
```cmd
docker exec gitlab-runner gitlab-runner --version
```

### View Available Commands
```cmd
docker exec gitlab-runner gitlab-runner --help
```

## Local Pipeline Execution

### Method 1: Using Docker Exec (Recommended)

Navigate to your project directory containing `.gitlab-ci.yml` and run:

```cmd
# Execute a specific job
docker exec -it --workdir /builds/project gitlab-runner gitlab-runner exec docker job_name

# Example: Run a job called "test"
docker exec -it gitlab-runner gitlab-runner exec docker test
```

### Method 2: Mount Project Directory

For easier access, you can mount your project directory:

```cmd
# Stop the existing container
docker stop gitlab-runner
docker rm gitlab-runner

# Create new container with project mounted
docker run -d --name gitlab-runner --restart always ^
  -v /var/run/docker.sock:/var/run/docker.sock ^
  -v gitlab-runner-config:/etc/gitlab-runner ^
  -v %cd%:/builds/project ^
  gitlab/gitlab-runner:latest
```

Then execute jobs:
```cmd
docker exec -it --workdir /builds/project gitlab-runner gitlab-runner exec docker test
```

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
```cmd
# Run the test job
docker exec -it --workdir /builds/project gitlab-runner gitlab-runner exec docker test

# Run the build job
docker exec -it --workdir /builds/project gitlab-runner gitlab-runner exec docker build
```

## Alternative: Using GitLab CI Local Tool

For a more user-friendly experience, you can use `gitlab-ci-local`:

### Install Node.js and npm
1. Download Node.js from [nodejs.org](https://nodejs.org/)
2. Install gitlab-ci-local:
```cmd
npm install -g gitlab-ci-local
```

### Run Pipeline Locally
```cmd
# In your project directory
gitlab-ci-local
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

### Pipeline Testing
```cmd
# List all jobs in .gitlab-ci.yml
docker exec --workdir /builds/project gitlab-runner gitlab-runner exec docker --list

# Run with specific Docker image
docker exec --workdir /builds/project gitlab-runner gitlab-runner exec docker --docker-image node:18 test

# Run with environment variables
docker exec --workdir /builds/project gitlab-runner gitlab-runner exec docker --env VAR_NAME=value test
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