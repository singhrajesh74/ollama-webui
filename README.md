# Ollama with WebUI Docker Setup

This repository (`singhrajesh74/ollama-webui`) uses GitHub Actions to build a Docker container for [Ollama](https://ollama.com/) with [Open WebUI](https://github.com/open-webui/open-webui) and deploy it to Docker Hub.

## Prerequisites
- GitHub account with repository `singhrajesh74/ollama-webui`.
- Docker Desktop installed locally.
- Git installed for repository management.
- Docker Hub account for pushing the image.

## Setup Instructions

### 1. Clone the Repository
```bash
git clone https://github.com/singhrajesh74/ollama-webui.git
cd ollama-webui
```

### 2. Create the Dockerfile
Create a `Dockerfile` in the repository root:

```dockerfile
FROM ollama/ollama:latest

RUN apt-get update && apt-get install -y \
    curl \
    && rm -rf /var/lib/apt/lists/*

RUN curl -fsSL https://deb.nodesource.com/setup_18.x | bash - && \
    apt-get install -y nodejs && \
    npm install -g @open-webui/open-webui

EXPOSE 11434 8080

CMD ["sh", "-c", "ollama serve & open-webui --port 8080"]
```

Commit and push:
```bash
git add Dockerfile
git commit -m "Add Dockerfile for Ollama with Open WebUI"
git push origin main
```

### 3. Configure GitHub Actions
Create `.github/workflows/build-docker.yml`:

```yaml
name: Build and Push Docker Image

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v6
        with:
          context: .
          file: Dockerfile
          push: true
          tags: singhrajesh74/ollama-webui:latest
```

Add Docker Hub secrets:
- Go to `https://github.com/singhrajesh74/ollama-webui/settings/secrets/actions`.
- Add `DOCKER_USERNAME` (your Docker Hub username) and `DOCKER_PASSWORD` (Docker Hub access token).

Commit and push:
```bash
mkdir -p .github/workflows
git add .github/workflows/build-docker.yml
git commit -m "Add GitHub Actions workflow for Docker build"
git push origin main
```

Check the workflow at `https://github.com/singhrajesh74/ollama-webui/actions`.

### 4. Run Locally
Pull the image:
```bash
docker pull singhrajesh74/ollama-webui:latest
```

Run the container:
```bash
docker run -d -p 11434:11434 -p 8080:8080 --name ollama-webui singhrajesh74/ollama-webui:latest
```

Access:
- WebUI: `http://localhost:8080`
- Ollama API: `http://localhost:11434`

### 5. Test and Troubleshoot
Test Ollama:
```bash
curl http://localhost:11434/api/version
```

Check WebUI logs if needed:
```bash
docker logs ollama-webui
```

**Common Issues**:
- **Port Conflicts**: Ensure ports 11434 and 8080 are free.
- **Build Failures**: Check GitHub Actions logs.
- **WebUI Errors**: Verify Open WebUI installation steps (see [Open WebUI GitHub](https://github.com/open-webui/open-webui)).

### 6. Optional Enhancements
- **Persistent Storage**:
  ```bash
  docker run -d -p 11434:11434 -p 8080:8080 -v ollama-data:/root/.ollama --name ollama-webui singhrajesh74/ollama-webui:latest
  ```
- **Environment Variables**: Add WebUI settings (e.g., `--env OPEN_WEBUI_SECRET_KEY=your-key`).
- **Local Test**:
  ```bash
  docker build -t ollama-webui .
  docker run -p 11434:11434 -p 8080:8080 ollama-webui
  ```

## Notes
- Use GitHub Container Registry by updating the workflow to push to `ghcr.io/singhrajesh74/ollama-webui:latest`.
- Ensure Docker Desktop is running (`docker ps`) before pulling or running containers.