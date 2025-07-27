# 📖 Installation Guide

Complete setup guide for the AI Web Extractor system with multi-agent workflows and MCP server integration.

## 📋 **Prerequisites**

### **System Requirements**
- **Operating System**: Linux, macOS, or Windows 10+
- **RAM**: 4GB minimum, 8GB recommended for optimal performance
- **Storage**: 15GB free space (includes Docker images and data)
- **Network**: Stable internet connection for API calls and container pulls

### **Required Software**

#### **Docker & Docker Compose**
```bash
# Install Docker (choose your platform)
# Linux (Ubuntu/Debian)
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# macOS - Download Docker Desktop from https://docs.docker.com/desktop/mac/
# Windows - Download Docker Desktop from https://docs.docker.com/desktop/windows/

# Verify installation
docker --version
docker-compose --version  # Should be 2.0+
```

#### **Git**
```bash
# Linux (Ubuntu/Debian)
sudo apt update && sudo apt install git

# macOS
brew install git

# Windows
# Download from https://git-scm.com/downloads

# Verify installation
git --version
```

### **Required API Keys**

#### **OpenAI API Key** (Required)
1. Visit [OpenAI Platform](https://platform.openai.com/api-keys)
2. Create account or sign in
3. Click **"Create new secret key"**
4. Copy and save the key (starts with `sk-`)
5. **Important**: Ensure sufficient credits for API usage

#### **Groq API Key** (Optional)
1. Visit [Groq Console](https://console.groq.com/keys)
2. Create account and verify email
3. Generate new API key
4. Copy and save the key (starts with `gsk_`)

## 🚀 **Installation Methods**

### **Method 1: Automated Setup (Recommended)**

#### **Step 1: Clone Repository**
```bash
# Clone the project
git clone https://github.com/your-username/ai-web-extractor.git
cd ai-web-extractor

# Verify repository structure
ls -la
```

#### **Step 2: Configure Environment**
```bash
# Copy environment template
cp .env.example .env

# Edit environment file
nano .env  # or use your preferred editor

# Required settings to configure:
OPENAI_API_KEY=sk-your-actual-key-here
N8N_PASSWORD=your-secure-password-here
```

#### **Step 3: Run Automated Setup**
```bash
# Make setup script executable
chmod +x scripts/setup-containers.sh

# Run automated setup
./scripts/setup-containers.sh

# The script will:
# - Create Docker network
# - Build MCP server image  
# - Start both containers
# - Test connectivity
# - Display access information
```

#### **Step 4: Verify Installation**
```bash
# Check container status
docker ps

# Expected output should show:
# - mcp-scraper (port 8000)
# - n8n_automation (port 5678)

# Test services
curl http://localhost:8000/health  # MCP server
curl http://localhost:5678/healthz # n8n
```

### **Method 2: Manual Setup**

#### **Step 1: Create Docker Network**
```bash
# Create shared network for containers
docker network create ai-extractor-network

# Verify network creation
docker network ls | grep ai-extractor
```

#### **Step 2: Build MCP Server**
```bash
# Navigate to MCP server directory
cd mcp-server

# Build Docker image
docker build -t mcp-scraper .

# Verify image creation
docker images | grep mcp-scraper
```

#### **Step 3: Start MCP Server**
```bash
# Run MCP server container
docker run -d \
  --name mcp-scraper \
  --network ai-extractor-network \
  -p 8000:8000 \
  -e MCP_LOG_LEVEL=info \
  mcp-scraper

# Check MCP server logs
docker logs mcp-scraper
```

#### **Step 4: Start n8n Container**
```bash
# Run n8n container (replace password)
docker run -d \
  --name n8n_automation \
  --network ai-extractor-network \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  -e N8N_BASIC_AUTH_ACTIVE=true \
  -e N8N_BASIC_AUTH_USER=admin \
  -e N8N_BASIC_AUTH_PASSWORD=your-secure-password \
  n8nio/n8n:latest

# Check n8n logs
docker logs n8n_automation
```

#### **Step 5: Test Container Communication**
```bash
# Test network connectivity
docker exec n8n_automation curl http://mcp-scraper:8000/health
docker exec n8n_automation wget http://mcp-scraper:8000/sse

# Should show successful connection to MCP server
```

## ⚙️ **Configuration**

### **Access Applications**

| Service | URL | Credentials |
|---------|-----|-------------|
| **n8n Interface** | http://localhost:5678 | admin / your-password |
| **MCP Server Health** | http://localhost:8000/health | No auth required |

### **Import Workflow**

#### **Step 1: Access n8n**
1. Open browser to `http://localhost:5678`
2. Login with credentials from your `.env` file
3. You should see the n8n workflow editor

#### **Step 2: Import Workflow File**
1. Click **"Import from file"** or press `Ctrl+O`
2. Navigate to `workflows/Ai_extractor_workflows.json`
3. Select file and click **"Open"**
4. Click **"Import"** to add workflow to n8n

#### **Step 3: Configure Credentials**

**OpenAI Credentials:**
1. Go to **Settings** → **Credentials**
2. Click **"Add Credential"**
3. Search for and select **"OpenAI"**
4. Enter your API key from earlier
5. Set credential name to: `OpenAi account` (exact match required)
6. Click **"Save"**

**Verify MCP Connection:**
1. Open the imported workflow
2. Find the **"MCP Client"** node
3. Verify settings:
   ```
   SSE Endpoint: http://mcp-scraper:8000/sse/
   Include Tools: ☑️ analyze_page_structure
   ```

### **Test Installation**

#### **Basic Functionality Test**
1. In n8n, open the imported workflow
2. Click **"Execute Workflow"** button
3. When prompted, enter a test URL: `https://example.com`
4. Monitor execution progress in the workflow editor
5. Check output in the final node for extracted data

#### **Container Health Test**
```bash
# Check all containers are running
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# Test MCP server endpoints
curl -s http://localhost:8000/health | jq .

# Test n8n health
curl -s http://localhost:5678/healthz
```

## 🔧 **Troubleshooting**

### **Container Issues**

**Containers not starting:**
```bash
# Check Docker daemon
sudo systemctl status docker  # Linux
# or restart Docker Desktop (Windows/Mac)

# Check available resources
docker system df
docker system prune  # Clean up if needed

# Check logs for specific errors
docker logs mcp-scraper
docker logs n8n_automation
```

**Port conflicts:**
```bash
# Check if ports are already in use
netstat -tlnp | grep :5678  # n8n port
netstat -tlnp | grep :8000  # MCP port

# If ports are occupied, stop conflicting services or use different ports
```

**Network communication issues:**
```bash
# Recreate network
docker network rm ai-extractor-network
docker network create ai-extractor-network

# Restart containers with network
docker stop mcp-scraper n8n_automation
docker rm mcp-scraper n8n_automation

# Recreate containers (use your setup commands)
```

### **API & Authentication Issues**

**OpenAI API errors:**
```bash
# Test API key directly
curl -H "Authorization: Bearer $OPENAI_API_KEY" \
  https://api.openai.com/v1/models

# Common issues:
# - Invalid API key format
# - Insufficient credits
# - Rate limiting (429 errors)
# - Billing issues
```

**n8n authentication problems:**
```bash
# Reset n8n password
docker exec -it n8n_automation n8n user-management:reset --email=admin@example.com

# Or recreate container with new password
docker stop n8n_automation
docker rm n8n_automation
# Run with new N8N_BASIC_AUTH_PASSWORD
```

### **MCP Server Issues**

**MCP server not responding:**
```bash
# Check MCP server logs
docker logs -f mcp-scraper

# Common issues:
# - Playwright browser installation failed
# - Python dependencies missing
# - Memory/resource constraints

# Rebuild MCP image
docker build --no-cache -t mcp-scraper mcp-server/
```

**Playwright browser issues:**
```bash
# If browser installation failed, rebuild with verbose output
docker build --no-cache --progress=plain -t mcp-scraper mcp-server/
```

## 🔒 **Security Considerations**

### **Production Deployment**

**Environment Security:**
```bash
# Use strong passwords
N8N_PASSWORD=$(openssl rand -base64 32)

# Restrict network access
# Only expose necessary ports
# Use firewall rules to limit access
```

**Container Security:**
```bash
# Run with resource limits
docker run -d \
  --name mcp-scraper \
  --memory=2g \
  --cpus=1.5 \
  --restart=unless-stopped \
  mcp-scraper
```

**API Key Management:**
- Store API keys in secure environment variables
- Use Docker secrets for production
- Rotate API keys regularly
- Monitor API usage and costs

## 📊 **Performance Optimization**

### **Resource Allocation**

**Docker Resources:**
```bash
# Increase Docker memory (Docker Desktop)
# Recommended: 4GB+ RAM allocation

# Monitor resource usage
docker stats mcp-scraper n8n_automation
```

**Container Optimization:**
```bash
# Production container limits
docker run -d \
  --name mcp-scraper \
  --memory=2g \
  --cpus=1.5 \
  --restart=unless-stopped \
  mcp-scraper
```

### **Monitoring Setup**

**Basic Monitoring:**
```bash
# Check container health
docker ps --filter "name=mcp-scraper" --filter "name=n8n_automation"

# Monitor logs
docker logs -f mcp-scraper &
docker logs -f n8n_automation &
```

## 🎯 **Next Steps**

After successful installation:

1. **📚 Read Usage Guide**: Check `workflows/README.md` for detailed workflow usage
2. **🎮 Run Test Extractions**: Try different types of websites (products, articles, companies)
3. **⚙️ Customize Configuration**: Adjust quality thresholds and retry logic
4. **📊 Monitor Performance**: Track success rates and processing times
5. **🔧 Explore Advanced Features**: Custom schemas, batch processing, integrations

## 🆘 **Getting Help**

**Documentation:**
- [Workflow Usage Guide](../workflows/README.md)
- [MCP Server Documentation](../mcp-server/README.md)
- [Troubleshooting Guide](./troubleshooting.md)

**Support Channels:**
- **GitHub Issues**: Report bugs and request features
- **Documentation**: Check all README files for detailed information
- **Container Logs**: Most issues can be diagnosed from Docker logs

## 📄 **Installation Checklist**

- [ ] Docker and Docker Compose installed
- [ ] Git installed and repository cloned
- [ ] OpenAI API key obtained and configured
- [ ] Environment variables set in `.env` file
- [ ] Docker containers built and running
- [ ] Network connectivity tested between containers
- [ ] n8n accessible at http://localhost:5678
- [ ] MCP server responding at http://localhost:8000
- [ ] Workflow imported into n8n
- [ ] OpenAI credentials configured in n8n
- [ ] Test extraction completed successfully

---

<div align="center">

**✅ Installation Complete! Ready to extract intelligence from the web.**

[🚀 Start Extracting](../workflows/README.md) • [🔧 Configure MCP](../mcp-server/README.md) • [⬆️ Back to Main](../README.md)

</div>
