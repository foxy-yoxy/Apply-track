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

### **Local Development**
```bash
# Install dependencies
pip install -r requirements.txt

# Install Playwright browsers
playwright install chromium

# Run server
python mcp_server.py
```

Server starts on `http://localhost:8000`

### **Docker Development**
```bash
# Build image
docker build -t mcp-scraper .

# Run container
docker run -p 8000:8000 mcp-scraper
```

### **Test the Server**
```bash
# Health check
curl http://localhost:8000/health

# Test extraction (if you add HTTP endpoints)
curl -X POST http://localhost:8000/analyze \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com"}'
```

## 🔧 **Configuration**

### **Environment Variables**
```env
# Server configuration
MCP_LOG_LEVEL=info          # Logging level (debug, info, warning, error)
MCP_MAX_RETRIES=3           # Maximum retry attempts
MCP_TIMEOUT=45              # Request timeout in seconds
MCP_USER_AGENT=Mozilla/5.0... # Browser user agent

# Performance tuning
MCP_CONCURRENT_LIMIT=5      # Max concurrent extractions
MCP_RATE_LIMIT=100          # Requests per minute
MCP_CACHE_TTL=3600          # Cache duration in seconds
```

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

### **Health Check Endpoint**
```bash
curl http://localhost:8000/health
# Response: {"status": "healthy", "version": "1.0.0"}
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

### **Common Issues**

**Browser not found:**
```bash
# Reinstall Playwright browsers
playwright install chromium
playwright install-deps chromium
```

**Memory issues:**
```bash
# Increase Docker memory limit or system resources
# Monitor with: docker stats
```

**Timeout errors:**
```env
# Increase timeout in environment
MCP_TIMEOUT=60
```

**Permission errors in Docker:**
```dockerfile
# Ensure non-root user (already configured)
USER mcpuser
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
│   n8n Workflow │────│ MCP Server       │────│ Target Website │
│                 │    │                  │    │                 │
│ - AI Agents     │    │ - Playwright     │    │ - Any webpage   │
│ - Validation    │    │ - Content        │    │ - JavaScript    │
│ - Retry Logic   │    │   Analysis       │    │ - Dynamic       │
└─────────────────┘    │ - Quality        │    │   Content       │
                       │   Scoring        │    └─────────────────┘
                       └──────────────────┘
```

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
