# 🕷️ MCP Web Scraper Server

> **Universal web content extraction server using Playwright and Model Context Protocol (MCP)**

High-performance web scraping server that provides intelligent content extraction with confidence scoring, structured data parsing, and quality validation for any webpage.

## 🎯 **Features**

### **🧠 Intelligent Analysis**
- **Universal page detection** - Automatically identifies product pages, articles, biographies, timelines
- **Confidence scoring** - Provides reliability metrics for extracted data
- **Semantic understanding** - Recognizes content patterns and structures

### **🔍 Comprehensive Extraction**
- **Structured data** - JSON-LD, microdata, and meta tag parsing
- **Content analysis** - Headings, paragraphs, links, images, contact information
- **Product-specific** - Prices, descriptions, vendor details, ratings
- **Social media** - Platform links and contact methods

### **🛡️ Robust & Reliable**
- **Error recovery** - Multiple retry strategies with exponential backoff
- **Browser automation** - Full JavaScript rendering with Playwright
- **Health monitoring** - Built-in health checks and status endpoints
- **Performance optimized** - Async processing and resource management

## 🏗️ **API Endpoints**

### **`analyze_page_structure`**
Comprehensive webpage analysis with intelligent content extraction.

**Input:**
```json
{
  "url": "https://example.com/product-page"
}
```

**Output:**
```json
{
  "status": "success",
  "pageType": "product",
  "confidence": 0.95,
  "data": {
    "productInfo": {
      "title": "Premium Wireless Headphones",
      "price": "€299.99",
      "description": "High-quality audio with noise cancellation...",
      "brand": "AudioTech",
      "rating": "4.8/5"
    },
    "metadata": {
      "og:title": "Premium Wireless Headphones",
      "og:price:amount": "299.99",
      "og:price:currency": "EUR"
    },
    "contentAreas": [...],
    "structuredData": [...],
    "contactInfo": {
      "emails": ["support@audiotech.com"],
      "phones": ["+33 1 23 45 67 89"]
    }
  },
  "extraction_stats": {
    "metadata_fields": 15,
    "content_areas": 8,
    "links_found": 25,
    "images_found": 12
  }
}
```

### **`extract_content_with_plan`**
Targeted extraction using CSS selectors from extraction plan.

**Input:**
```json
{
  "url": "https://example.com",
  "extraction_plan": {
    "titleSelectors": ["h1", ".product-title"],
    "priceSelectors": [".price", "[data-price]"],
    "descriptionSelectors": [".description", ".product-details"]
  }
}
```

### **`validate_extraction_quality`**
Quality assessment and confidence scoring for extracted data.

**Input:**
```json
{
  "url": "https://example.com",
  "extracted_data": { /* extracted content */ }
}
```

**Output:**
```json
{
  "status": "success",
  "validation": {
    "overallConfidence": 0.85,
    "fieldValidation": {
      "passport.id": {"valid": true, "confidence": 1.0},
      "headings": {"valid": true, "confidence": 0.9}
    },
    "issues": [],
    "suggestions": ["High quality extraction - proceed with confidence"]
  }
}
```

### **`get_page_screenshot`**
Visual debugging and monitoring with page screenshots.

**Input:**
```json
{
  "url": "https://example.com"
}
```

**Output:**
```json
{
  "status": "success",
  "screenshot_base64": "iVBORw0KGgoAAAANSUhEUgAA...",
  "viewport": {"width": 1280, "height": 720}
}
```

## 🚀 **Quick Start**

### **Docker Container Setup (Recommended)**
```bash
# Build the MCP server image
docker build -t mcp-scraper .

# Create Docker network (if not exists)
docker network create ai-extractor-network

# Run MCP server container
docker run -d \
  --name mcp-scraper \
  --network ai-extractor-network \
  -p 8000:8000 \
  mcp-scraper

# Run n8n container (separate)
docker run -d \
  --name n8n_automation \
  --network ai-extractor-network \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  n8nio/n8n:latest
```

### **Container Communication**
In this setup:
- **MCP Server**: Available at `http://mcp-scraper:8000/sse` (internal) and `http://localhost:8000` (external)
- **n8n Interface**: Available at `http://localhost:5678`
- **Network**: Both containers communicate via `ai-extractor-network`

### **Local Development**
```bash
# Install dependencies
pip install -r requirements.txt

# Install Playwright browsers
playwright install chromium

# Run server
python mcp_server.py
```

### **Test the Setup**
```bash

# Test container network communication (internal)
docker exec n8n_automation wget http://mcp-scraper:8000/sse

# Verify container network details
docker network inspect ai-extractor-network
```

## 🔧 **Configuration**

### **Docker Network Setup**
```bash
# Create shared network for containers
docker network create ai-extractor-network

# List networks to verify
docker network ls

# Inspect network
docker network inspect ai-extractor-network
```


### **Container Configuration**
```bash
# MCP Server container
docker run -d \
  --name mcp-scraper \
  --network ai-extractor-network \
  -p 8000:8000 \
  mcp-scraper

# n8n container connection to MCP
# In n8n MCP Client node, use: http://mcp-scraper:8000/sse/
```

### **n8n MCP Client Configuration**
In your n8n workflow MCP Client node:
```
SSE Endpoint: http://mcp-scraper:8000/sse/
Tools: analyze_page_structure, extract_content_with_plan
```

**Important**: Use the trailing slash `/sse/` for proper MCP protocol communication.

## 📊 **Page Type Detection**

The server automatically detects page types with confidence scoring:

| Page Type | Detection Criteria | Confidence Factors |
|-----------|-------------------|-------------------|
| **Product** | Price elements, cart buttons, product schemas | E-commerce indicators, structured data |
| **Article** | Article tags, author info, publish dates | Content structure, metadata |
| **Biography** | Bio keywords, career mentions, personal info | Biographical indicators |
| **Company** | About pages, founding info, corporate structure | Business indicators |
| **Timeline** | Chronological content, date sequences | Temporal patterns |

## 🔍 **Content Extraction Strategies**

### **Multi-Strategy Approach**
1. **Structured Data First** - JSON-LD, microdata, meta tags
2. **Semantic Selectors** - Content-aware CSS selection
3. **Pattern Recognition** - Text pattern matching for contacts, dates
4. **Fallback Extraction** - Generic content area identification


## 🐛 **Debugging & Monitoring**

### **Container Health Checks**
```bash
# Check MCP server health (external access)
curl http://localhost:8000/sse

# Check from n8n container (internal network)
docker exec n8n_automation wget http://mcp-scraper:8000/sse

# View container logs
docker logs mcp-scraper
docker logs n8n_automation

# Follow logs in real-time
docker logs -f mcp-scraper
```

### **Network Debugging**
```bash
# Test container connectivity
docker exec mcp-scraper ping n8n_automation
docker exec n8n_automation ping mcp-scraper

# Check network configuration and container IPs
docker network inspect ai-extractor-network

# View container network details (find IP addresses)
docker inspect mcp-scraper | grep -A 10 NetworkSettings
docker inspect n8n_automation | grep -A 10 NetworkSettings

# List all containers in network
docker network inspect ai-extractor-network | grep -A 5 -B 5 "Name"
```

### **Container Management**
```bash
# Restart containers
docker restart mcp-scraper
docker restart n8n_automation

# Stop containers
docker stop mcp-scraper n8n_automation

# Remove containers (keeps images and volumes)
docker rm mcp-scraper n8n_automation

# Rebuild MCP server
docker build -t mcp-scraper .
docker stop mcp-scraper
docker rm mcp-scraper
docker run -d --name mcp-scraper --network ai-extractor-network -p 8000:8000 mcp-scraper
```

## 🔧 **Troubleshooting**

### **Common Container Issues**

**MCP server not accessible from n8n:**
```bash
# Check if containers are on same network
docker inspect mcp-scraper | grep NetworkMode
docker inspect n8n_automation | grep NetworkMode

# Test connectivity (your actual container names)
docker exec n8n_automation wget http://mcp-scraper:8000/sse

```

**Container startup failures:**
```bash
# Check container logs
docker logs mcp-scraper

# Common issues:
# - Port already in use: docker ps -a
# - Network not found: docker network create ai-extractor-network
# - Image build failed: docker build -t mcp-scraper .
```

**n8n MCP Client connection issues:**
```bash
# Verify endpoint in n8n MCP Client node
# Correct: http://mcp-scraper:8000/sse/
# Wrong: http://localhost:8000/sse/ (won't work from container)

# Test MCP endpoint manually
docker exec n8n_automation wget -O- http://mcp-scraper:8000/sse/
```

**Network communication issues:**
```bash
# Recreate network with your containers
docker network rm ai-extractor-network
docker network create ai-extractor-network

# Restart containers with network
docker stop mcp-scraper n8n_automation
docker rm mcp-scraper n8n_automation

# Recreate containers (use your run commands)
docker run -d --name mcp-scraper --network ai-extractor-network -p 8000:8000 mcp-scraper
docker run -d --name n8n_automation --network ai-extractor-network -p 5678:5678 -v n8n_data:/home/node/.n8n n8nio/n8n:latest
```

### **n8n MCP Client Troubleshooting**

**Connection refused errors:**
- ✅ Verify endpoint: `http://mcp-scraper:8000/sse/` (not localhost)
- ✅ Check network: Both containers in `ai-extractor-network`  
- ✅ Test connectivity: `docker exec n8n_automation ping mcp-scraper`
- ✅ Verify IP resolution: Your test shows `172.19.0.3:8000` working

**Tool not found errors:**
- Verify available tools in MCP server logs: `docker logs mcp-scraper`
- Check tool names: `analyze_page_structure`, `extract_content_with_plan`
- Ensure MCP server is fully started before n8n connects

**Successful Connection Indicators:**
```bash
# This should work (your test confirmed it does):
docker exec n8n_automation wget http://mcp-scraper:8000/sse
# Output: Connecting to mcp-scraper:8000 (172.19.0.3:8000) ✅
```

### **Performance Optimization**

**For production deployment:**
```bash
# Run with resource limits (using your container names)
docker run -d \
  --name mcp-scraper \
  --network ai-extractor-network \
  --memory=2g \
  --cpus=1.5 \
  -p 8000:8000 \
  mcp-scraper

docker run -d \
  --name n8n_automation \
  --network ai-extractor-network \
  --memory=1g \
  --cpus=1.0 \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  n8nio/n8n:latest
```

## 🙏 **Acknowledgments**

- **[Playwright](https://playwright.dev/)** - Reliable browser automation
- **[FastMCP](https://github.com/jlowin/fastmcp)** - Model Context Protocol implementation
- **[n8n](https://n8n.io)** - Workflow automation platform integration

---
