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
  -e MCP_LOG_LEVEL=info \
  mcp-scraper

# Run n8n container (separate)
docker run -d \
  --name n8n_automation \
  --network ai-extractor-network \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  -e N8N_BASIC_AUTH_ACTIVE=true \
  -e N8N_BASIC_AUTH_USER=admin \
  -e N8N_BASIC_AUTH_PASSWORD=your-password \
  n8nio/n8n:latest
```

### **Container Communication**
In this setup:
- **MCP Server**: Available at `http://mcp-scraper:8000` (internal) and `http://localhost:8000` (external)
- **n8n Interface**: Available at `http://localhost:5678`
- **Network**: Both containers communicate via `ai-extractor-network`
- **Container IPs**: Automatically assigned (e.g., `172.19.0.3` for MCP server)

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
# Health check MCP server (external)
curl http://localhost:8000/health

# Check n8n connectivity (external)
curl http://localhost:5678/healthz

# Test container network communication (internal)
docker exec n8n_automation curl http://mcp-scraper:8000/health
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

### **Environment Variables**
```env
# Server configuration
MCP_LOG_LEVEL=info          # Logging level (debug, info, warning, error)
MCP_MAX_RETRIES=3           # Maximum retry attempts
MCP_TIMEOUT=45              # Request timeout in seconds
MCP_USER_AGENT=Mozilla/5.0... # Browser user agent
HOST=0.0.0.0               # Bind to all interfaces for container access
PORT=8000                  # Internal container port

# Performance tuning
MCP_CONCURRENT_LIMIT=5      # Max concurrent extractions
MCP_RATE_LIMIT=100          # Requests per minute
MCP_CACHE_TTL=3600          # Cache duration in seconds
```

### **Container Configuration**
```bash
# MCP Server container
docker run -d \
  --name mcp-scraper \
  --network ai-extractor-network \
  -p 8000:8000 \
  -e MCP_LOG_LEVEL=info \
  -e MCP_TIMEOUT=45 \
  -e HOST=0.0.0.0 \
  -e PORT=8000 \
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

### **Playwright Configuration**
```python
# Browser settings (in code)
browser_config = {
    'headless': True,
    'viewport': {'width': 1920, 'height': 1080},
    'user_agent': 'Mozilla/5.0...',
    'timeout': 45000
}
```

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

### **Quality Indicators**
- **High Confidence (0.8+)**: Rich metadata, structured data, clear content areas
- **Medium Confidence (0.5-0.8)**: Some structured data, identifiable content
- **Low Confidence (0.3-0.5)**: Basic content only, minimal structure
- **Poor Quality (<0.3)**: Minimal extractable content

## 🐛 **Debugging & Monitoring**

### **Container Health Checks**
```bash
# Check MCP server health (external access)
curl http://localhost:8000/health

# Check from n8n container (internal network)
docker exec n8n_automation curl http://mcp-scraper:8000/health
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

### **Screenshot Debugging**
Use the `get_page_screenshot` tool to visually debug extraction issues:
```python
# In n8n workflow or direct API call
screenshot_result = await mcp.get_page_screenshot("https://problematic-site.com")
# Save screenshot_result.screenshot_base64 to debug visually
```

### **Logging Levels**
```python
# Set log level via environment
export MCP_LOG_LEVEL=debug

# Available levels: debug, info, warning, error
```

## 🔧 **Troubleshooting**

### **Common Container Issues**

**MCP server not accessible from n8n:**
```bash
# Check if containers are on same network
docker inspect mcp-scraper | grep NetworkMode
docker inspect n8n_automation | grep NetworkMode

# Test connectivity (your actual container names)
docker exec n8n_automation curl http://mcp-scraper:8000/health
docker exec n8n_automation wget http://mcp-scraper:8000/sse

# Verify network communication with IP
docker exec n8n_automation curl http://172.19.0.3:8000/health  # Use actual IP
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
  -e MCP_LOG_LEVEL=warning \
  -e MCP_CONCURRENT_LIMIT=10 \
  mcp-scraper

docker run -d \
  --name n8n_automation \
  --network ai-extractor-network \
  --memory=1g \
  --cpus=1.0 \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  -e N8N_BASIC_AUTH_ACTIVE=true \
  -e N8N_BASIC_AUTH_USER=admin \
  -e N8N_BASIC_AUTH_PASSWORD=your-secure-password \
  n8nio/n8n:latest
```

### **Performance Optimization**

**For high-volume usage:**
```env
# Increase concurrent limits
MCP_CONCURRENT_LIMIT=10
MCP_RATE_LIMIT=200

# Enable caching
MCP_CACHE_ENABLED=true
MCP_CACHE_TTL=3600
```

**Memory optimization:**
```python
# Browser resource limits (in code)
browser_args = [
    '--max_old_space_size=1024',
    '--no-sandbox',
    '--disable-dev-shm-usage'
]
```

## 📈 **Performance Metrics**

### **Typical Performance**
- **Average extraction time**: 2-5 seconds
- **Success rate**: 95%+ on standard websites
- **Memory usage**: ~200MB per concurrent extraction
- **CPU usage**: ~30% during active scraping

### **Optimization Tips**
- Use headless mode for production
- Implement request caching for repeated URLs
- Monitor memory usage with concurrent extractions
- Set appropriate timeouts for different site types

## 🏗️ **Architecture**

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│  n8n_automation │────│   mcp-scraper    │────│ Target Website │
│   (port 5678)   │    │   (port 8000)    │    │                 │
│                 │    │ IP: 172.19.0.3   │    │ - Any webpage   │
│ - AI Agents     │    │                  │    │ - JavaScript    │
│ - Validation    │    │ - Playwright     │    │ - Dynamic       │
│ - Retry Logic   │    │ - Content        │    │   Content       │
│                 │    │   Analysis       │    └─────────────────┘
└─────────────────┘    │ - Quality        │    
        │              │   Scoring        │    
        └─────────────────────┬─────────────────────┘
                ai-extractor-network
            (Container communication via hostnames)
```

### **Container Communication Flow**
1. **n8n Workflow** triggers MCP Client node
2. **MCP Client** sends request to `http://mcp-scraper:8000/sse/`
3. **Docker Network** resolves `mcp-scraper` to IP `172.19.0.3`
4. **MCP Server** receives request and processes with Playwright
5. **Playwright** loads target webpage with full browser
6. **Content Analysis** extracts structured data with confidence scoring
7. **Response** returns to n8n via container network
8. **n8n Agents** process the extracted data for validation

### **Network Configuration**
- **Network Name**: `ai-extractor-network` (default bridge)
- **Container Names**: `mcp-scraper`, `n8n_automation`
- **Internal Communication**: Container-to-container via hostnames
- **External Access**: Host ports (5678 for n8n, 8000 for MCP)
- **IP Range**: `172.19.0.x` subnet (Docker default)
- **Security**: Isolated network, containers only accessible to each other

## 🤝 **Contributing**

1. Fork the repository
2. Create a feature branch
3. Add tests for new functionality
4. Ensure all tests pass
5. Submit a pull request

### **Development Setup**
```bash
# Install development dependencies
pip install -r requirements.txt
pip install pytest pytest-asyncio black flake8

# Run tests
pytest

# Format code
black mcp_server.py

# Lint code
flake8 mcp_server.py
```

## 📄 **License**

This project is part of the AI Web Extractor system and follows the same MIT License.

## 🙏 **Acknowledgments**

- **[Playwright](https://playwright.dev/)** - Reliable browser automation
- **[FastMCP](https://github.com/jlowin/fastmcp)** - Model Context Protocol implementation
- **[n8n](https://n8n.io)** - Workflow automation platform integration

---

<div align="center">

**🚀 Ready to extract intelligence from any webpage!**

[⬆️ Back to Main Project](../README.md) • [📖 Documentation](../docs/) • [🐛 Issues](../../issues)

</div>
