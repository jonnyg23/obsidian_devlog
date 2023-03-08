# Docker

> Containerizes application.

## Commands

### Stop all `in-use` docker images

> NOTE: These commands were performed in Windows. They may or may not be different with Linux and MacOS.

```bash
# Stop in-use images
docker rm -vf $(docker ps -aq)

# Remove images
docker rmi -f $(docker images -aq)
```

## Workflow

### Check to see if docker is installed & what version you have.

```bash
# Checks docker version
docker -v
```
