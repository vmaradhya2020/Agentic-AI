# Docker Deployment Guide

Complete guide for deploying the GenAI Investment Advisor using Docker.

---

## Prerequisites

- Docker installed (version 20.10+)
- Docker Compose installed (version 1.29+)
- API Keys:
  - Finnhub API Key (get free at https://finnhub.io/)
  - Google Gemini API Key (get at https://ai.google.dev/)

---

## Quick Start

### 1. Set Up Environment Variables

Create a `.env` file in the project root:

```bash
FINNHUB_API_KEY=your_finnhub_api_key_here
GEMINI_API_KEY=your_gemini_api_key_here
```

**Security Note:** Never commit `.env` file to version control!

---

### 2. Build and Run with Docker Compose (Recommended)

```bash
# Build and start the container
docker-compose up -d

# View logs
docker-compose logs -f

# Stop the container
docker-compose down
```

The application will be available at: **http://localhost:7860**

---

### 3. Build and Run with Docker (Alternative)

```bash
# Build the image
docker build -t genai-investment-advisor .

# Run the container
docker run -d \
  -p 7860:7860 \
  --env-file .env \
  --name stock-analyzer \
  genai-investment-advisor

# View logs
docker logs -f stock-analyzer

# Stop and remove
docker stop stock-analyzer
docker rm stock-analyzer
```

---

## Docker Commands Reference

### Container Management

```bash
# Start the container
docker-compose start

# Stop the container
docker-compose stop

# Restart the container
docker-compose restart

# View running containers
docker ps

# View all containers (including stopped)
docker ps -a
```

### Logs and Debugging

```bash
# View logs
docker-compose logs

# Follow logs in real-time
docker-compose logs -f

# View last 100 lines
docker-compose logs --tail=100

# Execute commands inside container
docker-compose exec genai-investment-advisor bash

# View container resource usage
docker stats stock-analyzer
```

### Cleanup

```bash
# Stop and remove containers
docker-compose down

# Remove containers and volumes
docker-compose down -v

# Remove images
docker rmi genai-investment-advisor

# Clean up everything (use with caution!)
docker system prune -a
```

---

## Production Deployment

### Environment Variables

For production, use Docker secrets or environment variable injection:

```bash
# Using environment variables directly
docker run -d \
  -p 7860:7860 \
  -e FINNHUB_API_KEY="your_key" \
  -e GEMINI_API_KEY="your_key" \
  --name stock-analyzer \
  genai-investment-advisor
```

### With Docker Secrets (Swarm Mode)

```bash
# Create secrets
echo "your_finnhub_key" | docker secret create finnhub_key -
echo "your_gemini_key" | docker secret create gemini_key -

# Deploy with secrets
docker service create \
  --name stock-analyzer \
  --secret finnhub_key \
  --secret gemini_key \
  -p 7860:7860 \
  genai-investment-advisor
```

---

## Cloud Deployment

### Deploy to AWS ECS

1. **Build and push to ECR:**
```bash
# Authenticate to ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin YOUR_ACCOUNT.dkr.ecr.us-east-1.amazonaws.com

# Tag image
docker tag genai-investment-advisor:latest YOUR_ACCOUNT.dkr.ecr.us-east-1.amazonaws.com/genai-investment-advisor:latest

# Push image
docker push YOUR_ACCOUNT.dkr.ecr.us-east-1.amazonaws.com/genai-investment-advisor:latest
```

2. **Create ECS task definition and service** (use AWS Console or CLI)

### Deploy to Google Cloud Run

```bash
# Build and push to GCR
gcloud builds submit --tag gcr.io/YOUR_PROJECT_ID/genai-investment-advisor

# Deploy to Cloud Run
gcloud run deploy genai-investment-advisor \
  --image gcr.io/YOUR_PROJECT_ID/genai-investment-advisor \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --set-env-vars FINNHUB_API_KEY=your_key,GEMINI_API_KEY=your_key
```

### Deploy to Azure Container Instances

```bash
# Login to Azure
az login

# Create resource group
az group create --name genai-rg --location eastus

# Deploy container
az container create \
  --resource-group genai-rg \
  --name stock-analyzer \
  --image genai-investment-advisor \
  --dns-name-label stock-analyzer-unique \
  --ports 7860 \
  --environment-variables \
    FINNHUB_API_KEY=your_key \
    GEMINI_API_KEY=your_key
```

---

## Troubleshooting

### Container Won't Start

```bash
# Check logs
docker-compose logs

# Common issues:
# 1. Missing API keys in .env
# 2. Port 7860 already in use
# 3. Insufficient memory
```

### Port Already in Use

```bash
# Find process using port 7860
lsof -i :7860  # Mac/Linux
netstat -ano | findstr :7860  # Windows

# Change port in docker-compose.yml
ports:
  - "8080:7860"  # Use port 8080 instead
```

### API Key Issues

```bash
# Verify .env file exists and has correct format
cat .env

# Ensure no spaces around = sign
# Correct:   FINNHUB_API_KEY=abc123
# Incorrect: FINNHUB_API_KEY = abc123
```

### Permission Denied (Linux)

```bash
# Run docker commands with sudo
sudo docker-compose up -d

# Or add user to docker group
sudo usermod -aG docker $USER
# Log out and back in for changes to take effect
```

---

## Performance Optimization

### Multi-Stage Build (Smaller Image)

Create `Dockerfile.optimized`:

```dockerfile
# Build stage
FROM python:3.12-slim as builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Runtime stage
FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY mcp_client.py mcp_server.py ./
ENV PATH=/root/.local/bin:$PATH
EXPOSE 7860
CMD ["python", "mcp_client.py"]
```

Build with:
```bash
docker build -f Dockerfile.optimized -t genai-investment-advisor:optimized .
```

### Resource Limits

```yaml
# In docker-compose.yml
services:
  genai-investment-advisor:
    # ... other config ...
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 2G
        reservations:
          cpus: '1.0'
          memory: 1G
```

---

## Monitoring

### Health Check

The Dockerfile includes a health check. View status:

```bash
docker inspect --format='{{.State.Health.Status}}' stock-analyzer
```

### Container Metrics

```bash
# Real-time resource usage
docker stats stock-analyzer

# Export metrics to file
docker stats --no-stream stock-analyzer > metrics.txt
```

---

## Security Best Practices

1. **Never commit `.env` to version control**
   - Add `.env` to `.gitignore`

2. **Use secrets management in production**
   - AWS Secrets Manager
   - Google Secret Manager
   - Azure Key Vault

3. **Run as non-root user** (add to Dockerfile):
   ```dockerfile
   RUN useradd -m -u 1000 appuser
   USER appuser
   ```

4. **Keep base image updated**
   ```bash
   docker pull python:3.12-slim
   docker-compose build --no-cache
   ```

5. **Scan for vulnerabilities**
   ```bash
   docker scan genai-investment-advisor
   ```

---

## FAQ

**Q: Can I use this with Kubernetes?**
A: Yes! Convert docker-compose to K8s manifests using `kompose`:
```bash
kompose convert
kubectl apply -f .
```

**Q: How do I update the application?**
A: Rebuild and restart:
```bash
docker-compose build --no-cache
docker-compose up -d
```

**Q: Can I access from other devices on my network?**
A: Yes, use your machine's IP instead of localhost:
```
http://192.168.1.x:7860
```

**Q: How do I backup data?**
A: The app is stateless. Just backup your `.env` file and source code.

---

## Support

For issues, refer to:
- Project README: `README.md`
- Docker docs: https://docs.docker.com
- Docker Compose docs: https://docs.docker.com/compose/

---

**Built with Docker ❤️**
