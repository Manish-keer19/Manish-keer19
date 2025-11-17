# Essential Docker and CI/CD Cheatsheet

A comprehensive guide to Docker commands, Docker Compose, and GitHub Actions CI/CD workflows.

---

## I. Dockerfile Instructions

Understanding Dockerfile instructions is essential for building Docker images. Each instruction creates a layer in your image.

| Instruction | Description | Example |
|-------------|-------------|---------|
| `FROM` | Specifies the base image to build upon | `FROM node:18-alpine` |
| `WORKDIR` | Sets the working directory inside the container | `WORKDIR /app` |
| `COPY` | Copies files/directories from host to container | `COPY package*.json ./` |
| `ADD` | Similar to COPY but supports URLs and auto-extraction of archives | `ADD archive.tar.gz /app/` |
| `RUN` | Executes commands during the build process (installs packages, etc.) | `RUN npm install` |
| `CMD` | Defines the default command to run when container starts | `CMD ["npm", "start"]` |
| `ENTRYPOINT` | Configures container to run as an executable | `ENTRYPOINT ["node", "server.js"]` |
| `EXPOSE` | Documents which ports the container will listen on | `EXPOSE 3000` |
| `ENV` | Sets environment variables | `ENV NODE_ENV=production` |
| `ARG` | Defines build-time variables | `ARG VERSION=1.0` |
| `VOLUME` | Creates a mount point for persistent data | `VOLUME /app/data` |
| `USER` | Sets the user for running subsequent instructions | `USER node` |
| `LABEL` | Adds metadata to the image | `LABEL version="1.0" maintainer="you@example.com"` |

### Sample Dockerfile for Node.js Application

```dockerfile
# Use official Node.js image as base
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files first (for better caching)
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy application code
COPY . .

# Expose the application port
EXPOSE 3000

# Define the command to run the app
CMD ["npm", "start"]
```

---

## II. Essential Docker Commands

### Image Commands

| Command | Description | Example |
|---------|-------------|---------|
| `docker build` | Build an image from a Dockerfile | `docker build -t myapp:v1 .` |
| `docker images` | List all local images | `docker images` or `docker images -a` (all) |
| `docker pull` | Download an image from a registry | `docker pull nginx:latest` |
| `docker push` | Upload an image to a registry | `docker push username/myapp:latest` |
| `docker rmi` | Remove one or more images | `docker rmi myapp:v1` or `docker rmi -f myapp:v1` (force) |
| `docker tag` | Create a tag for an image | `docker tag myapp:v1 username/myapp:latest` |
| `docker history` | Show the history of an image | `docker history myapp:v1` |
| `docker save` | Save an image to a tar archive | `docker save -o myapp.tar myapp:v1` |
| `docker load` | Load an image from a tar archive | `docker load -i myapp.tar` |

### Container Commands

| Command | Description | Example |
|---------|-------------|---------|
| `docker run` | Create and start a container from an image | `docker run -d -p 8080:80 --name mycontainer nginx` |
| `docker ps` | List running containers | `docker ps` or `docker ps -a` (all containers) |
| `docker start` | Start one or more stopped containers | `docker start mycontainer` |
| `docker stop` | Stop one or more running containers | `docker stop mycontainer` |
| `docker restart` | Restart one or more containers | `docker restart mycontainer` |
| `docker rm` | Remove one or more containers | `docker rm mycontainer` or `docker rm -f mycontainer` (force) |
| `docker logs` | Fetch logs from a container | `docker logs mycontainer` or `docker logs -f mycontainer` (follow) |
| `docker exec` | Execute a command inside a running container | `docker exec -it mycontainer bash` |
| `docker attach` | Attach to a running container | `docker attach mycontainer` |
| `docker cp` | Copy files between container and host | `docker cp mycontainer:/app/file.txt ./file.txt` |
| `docker inspect` | Display detailed information about a container | `docker inspect mycontainer` |
| `docker stats` | Display live resource usage statistics | `docker stats` or `docker stats mycontainer` |
| `docker port` | List port mappings for a container | `docker port mycontainer` |

### System & Utility Commands

| Command | Description | Example |
|---------|-------------|---------|
| `docker version` | Show Docker version information | `docker version` |
| `docker info` | Display system-wide information | `docker info` |
| `docker network ls` | List all Docker networks | `docker network ls` |
| `docker volume ls` | List all Docker volumes | `docker volume ls` |
| `docker system df` | Show Docker disk usage | `docker system df` |
| `docker system prune` | Remove unused data | `docker system prune -a` |

---

## III. Docker Compose Basics

Docker Compose simplifies managing multi-container applications using a `docker-compose.yml` file.

### Key Commands

| Command | Description | Example |
|---------|-------------|---------|
| `docker-compose up` | Create and start all services defined in docker-compose.yml | `docker-compose up -d` (detached mode) |
| `docker-compose down` | Stop and remove all containers, networks, and volumes | `docker-compose down` |
| `docker-compose logs` | View output from all services | `docker-compose logs -f` (follow logs) |
| `docker-compose ps` | List containers managed by docker-compose | `docker-compose ps` |
| `docker-compose build` | Build or rebuild services | `docker-compose build` |
| `docker-compose restart` | Restart services | `docker-compose restart` |

### Sample docker-compose.yml

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8080:80"
    environment:
      - NODE_ENV=production
    depends_on:
      - db
  
  db:
    image: postgres:14
    environment:
      - POSTGRES_PASSWORD=mysecretpassword
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

---

## IV. GitHub Actions CI/CD for Docker

This workflow automates building and pushing Docker images to Docker Hub whenever code is pushed to the `main` branch.

### Prerequisites

1. Create a Docker Hub account at https://hub.docker.com
2. In your GitHub repository, go to **Settings → Secrets and variables → Actions**
3. Add the following secrets:
   - `DOCKERHUB_USERNAME`: Your Docker Hub username
   - `DOCKERHUB_TOKEN`: Your Docker Hub access token (create at Docker Hub → Account Settings → Security)

### GitHub Actions Workflow

Create this file in your repository: `.github/workflows/docker-publish.yml`

```yaml
name: Build and Push Docker Image

# Trigger the workflow on push events to the main branch
on:
  push:
    branches:
      - main

# Define the jobs to be executed
jobs:
  build-and-push:
    # Run this job on the latest Ubuntu runner
    runs-on: ubuntu-latest
    
    steps:
      # Step 1: Check out the repository code
      - name: Checkout code
        uses: actions/checkout@v4
      
      # Step 2: Set up Docker Buildx (advanced build features)
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      # Step 3: Log in to Docker Hub using secrets
      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      
      # Step 4: Extract metadata (tags, labels) for Docker
      - name: Extract Docker metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          # Replace 'username/repository' with your Docker Hub username and repo name
          images: username/repository
          tags: |
            # Tag with 'latest' for main branch
            type=raw,value=latest,enable={{is_default_branch}}
            # Tag with git commit SHA
            type=sha,prefix={{branch}}-
      
      # Step 5: Build and push Docker image
      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          # Build context is the current directory
          context: .
          # Path to your Dockerfile (default is ./Dockerfile)
          file: ./Dockerfile
          # Push the image to Docker Hub
          push: true
          # Use the tags generated from the metadata step
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          # Enable build cache for faster builds
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

### Customization Tips

- **Change the image name**: Replace `username/repository` in the workflow with your actual Docker Hub username and repository name (e.g., `johndoe/myapp`)
- **Add multiple tags**: Modify the `tags` section in the metadata step to include version numbers, branch names, etc.
- **Build from different Dockerfile**: Change the `file` parameter to point to a different Dockerfile location
- **Add build arguments**: Include `build-args` in the build-push step to pass variables to your Dockerfile

### Workflow Breakdown

1. **Trigger**: Activates on every push to the `main` branch
2. **Checkout**: Retrieves your repository code
3. **Buildx Setup**: Prepares Docker for advanced building features
4. **Login**: Authenticates with Docker Hub using stored secrets
5. **Metadata**: Generates appropriate tags (latest, commit SHA)
6. **Build & Push**: Compiles the Docker image and uploads it to Docker Hub

---

## Quick Reference

### Common Docker Run Flags

- `-d`: Run container in detached mode (background)
- `-p HOST:CONTAINER`: Map port from host to container
- `--name`: Assign a name to the container
- `-e KEY=VALUE`: Set environment variables
- `-v HOST:CONTAINER`: Mount volume from host to container
- `--rm`: Automatically remove container when it stops
- `-it`: Interactive terminal mode

### Cleaning Up Docker

```bash
# Remove all stopped containers
docker container prune

# Remove all unused images
docker image prune -a

# Remove all unused volumes
docker volume prune

# Remove everything (containers, images, volumes, networks)
docker system prune -a --volumes
```

---

## Additional Resources

- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Docker Hub](https://hub.docker.com/)