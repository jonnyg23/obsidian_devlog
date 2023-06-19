# Docker

> Containerizes application to run 

## Commands

### Stop all `in-use` docker images

> NOTE: These commands were performed in Windows 10. They may or may not be different with Linux and MacOS.

```bash
# Stop in-use images
docker rm -vf $(docker ps -aq)

# Remove images
docker rmi -f $(docker images -aq)
```

## Workflow


```bash
# Checks docker version
docker -v

# Navigate to project
cd /path/to/project

# Build the Docker Image
docker build -t myimage:latest .

# Run Docker container
docker run -d --name mycontainer myimage:latest
```

