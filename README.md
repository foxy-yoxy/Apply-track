# 🔄 AI Web Extraction Workflows

This directory contains n8n workflow definitions for intelligent web data extraction with multi-agent AI validation.

## 📁 **Directory Structure**

```
workflows/
├── README.md                           # This file
├── Ai_extractor_workflows.json         # Main extraction workflow
```

## 🎯 **Main Workflow: Ai_extractor_workflows.json**

### **Overview**
Multi-agent AI pipeline that transforms any webpage URL into structured JSON data with quality validation and intelligent retry logic.

### **Workflow Components**

#### **1. Input & Validation Layer**
- **When chat message received** - Chat trigger for URL input
- **Validate URL** - JavaScript code node for URL format validation
- **If** - Routes valid URLs forward, rejects invalid ones
- **Check Url accessibility** - HTTP HEAD request to test URL reachability
- **IF** - Checks HTTP response status (200 = accessible)
- **Url Error Handler** - Handles inaccessible or invalid URLs

#### **2. AI Agent Pipeline**
- **Planner Agent** (OpenAI GPT-4 Turbo) - Analyzes webpage with MCP tools
  - Uses MCP Client to call `analyze_page_structure`
  - Extracts comprehensive page information
  - Provides intelligent reasoning about content
- **Extractor Agent** (OpenAI GPT-4 Turbo) - Transforms analysis into JSON
  - Maps analysis to structured JSON schema
  - Uses logical deduction for missing fields
  - Applies business rules (e.g., .fr domain = France)
- **Validator Agent** (OpenAI GPT-4) - Quality assessment and scoring
  - Counts filled vs empty fields dynamically
  - Calculates confidence and completeness scores
  - Provides actionable improvement suggestions

#### **3. Processing & Control Layer**
- **Code** - JSON Parser that cleans AI-generated output
  - Removes markdown formatting and escape characters
  - Provides both individual fields and full JSON string
  - Handles malformed JSON with error recovery
- **Parser validation code** - Extracts validation metrics
  - Robust JSON extraction with 3-tier fallback
  - Returns structured validation data
- **Gate quality** - Quality-based decision logic
  - Implements retry mechanism for low-quality extractions
  - Sets thresholds (40% completeness, 30% confidence)
  - Maximum 2 retry attempts with enhanced prompts

#### **4. Output Layer**
- **Parser** - Final JSON cleaner for output data
- **LayoutData** - Template mapper for UI display format

### **Data Flow**
```
URL Input → Validation → Accessibility → Planner → Extractor → Validator → Quality Gate → Final Output
    ↓           ↓           ↓           ↓          ↓          ↓           ↓
  Chat UI   Format      HTTP Test   MCP Tools   JSON     Confidence  Template
           Check                              Structure  Scoring     Mapping
                                                  ↓
                                            Retry Logic (max 2 attempts)
```

### **MCP Integration**
The workflow connects to your MCP server container:
- **SSE Endpoint**: `http://mcp-scraper:8000/sse/`
- **Tools Used**: `analyze_page_structure`
- **Container Communication**: Via Docker network `ai-extractor-network`

## 🔧 **Import Instructions**

### **Step 1: Import Workflow**
1. Open n8n interface at `http://localhost:5678`
2. Login with your credentials
3. Click **"Import from file"** or use `Ctrl+O`
4. Select `Ai_extractor_workflows.json`
5. Click **"Import"**

### **Step 2: Configure Credentials**
The workflow requires OpenAI API credentials:

**OpenAI Credential Setup:**
1. Go to **Credentials** → **Add Credential**
2. Select **"OpenAI"**
3. Enter your API key
4. Name it exactly: `OpenAi account` (matches workflow expectation)
5. Save

### **Step 3: Configure MCP Connection**
Verify the **MCP Client** node settings:
```
SSE Endpoint: http://mcp-scraper:8000/sse/
Include Tools: ☑️ analyze_page_structure
```

**Important**: Ensure your MCP server container is running and accessible.

### **Step 4: Test Workflow**
1. Click **"Execute Workflow"** or use the chat interface
2. Enter test URL: `https://example.com/product-page`
3. Monitor execution in real-time
4. Check output quality and structure

## 📊 **Workflow Performance**

### **Typical Metrics**
- **Processing Time**: 20-45 seconds per URL
- **Success Rate**: 85-95% depending on site complexity
- **Quality Score**: 0.6-0.9 confidence range
- **Retry Rate**: 15-20% of extractions trigger retry

### **Quality Thr## 🚀 **Usage Examples**

### **Product Page Extraction**
```bash
# Input URL
https://shop.example.com/wireless-headphones

# Expected Output
{
  "passport": {
    "id": "https://shop.example.com/wireless-headphones",
    "authority": "shop.example.com",
    "issuedOn": "2025-01-15T10:30:00Z"
  },
  "steps": [{
    "narrative": {
      "title": "Premium Wireless Headphones",
      "paragraph": "High-quality audio with noise cancellation..."
    },
    "extraNote": "€299.99"
  }],
  "actors": [{
    "fullName": "AudioTech Solutions",
    "country": "France",
    "role": "vendor"
  }]
}
```

### **Article Page Extraction**
```bash
# Input URL
https://news.example.com/ai-breakthrough

# Expected Output
{
  "passport": {
    "id": "https://news.example.com/ai-breakthrough",
    "authority": "news.example.com"
  },
  "steps": [{
    "narrative": {
      "title": "AI Breakthrough in Medical Diagnosis",
      "paragraph": "Researchers have developed a new AI system..."
    }
  }],
  "actors": [{
    "fullName": "News Example Media",
    "role": "publisher"
  }]
}
```

## 🔍 **Troubleshooting**

### **Common Issues**

**MCP Connection Failed:**
```bash
# Check MCP server status
docker logs mcp-scraper

# Test connectivity from n8n container
docker exec n8n_automation curl http://mcp-scraper:8000/health

# Verify endpoint in MCP Client node
SSE Endpoint: http://mcp-scraper:8000/sse/  # Note the trailing slash
```

**OpenAI API Errors:**
```bash
# Check credential configuration
- Credential name must be exactly: "OpenAi account"
- Verify API key is valid and has sufficient credits
- Check rate limits if getting 429 errors
```

**Low Quality Extractions:**
```bash
# Common causes:
- JavaScript-heavy websites (MCP should handle this)
- Blocked by CORS or anti-bot measures
- Content behind authentication
- Very minimal content on page

# Solutions:
- Let retry logic attempt enhanced extraction
- Check MCP server logs for specific errors
- Test with simpler websites first
```

**Workflow Execution Timeout:**
```bash
# Typical processing times:
- Simple pages: 15-25 seconds
- Complex pages: 30-45 seconds
- With retries: 60-90 seconds

# If timing out consistently:
- Check MCP server performance
- Verify Docker container resources
- Monitor OpenAI API response times
```

### **Performance Optimization**

**For High-Volume Usage:**
- Use webhooks instead of chat trigger for batch processing
- Implement caching for frequently accessed URLs
- Monitor OpenAI API usage and costs
- Consider using Groq for validation to reduce OpenAI costs

## 📊 **Monitoring & Analytics**

### **Key Metrics to Track**
- **Success Rate**: % of successful extractions
- **Quality Score**: Average completeness and confidence
- **Processing Time**: Average time per extraction
- **Retry Rate**: % of extractions requiring retries
- **API Costs**: OpenAI token usage per extraction

### **Quality Indicators**
- **High Quality (0.8+ completeness)**: Rich structured data
- **Medium Quality (0.5-0.8)**: Basic information extracted
- **Low Quality (0.3-0.5)**: Minimal data, may need manual review
- **Poor Quality (<0.3)**: Failed extraction, manual intervention needed

## 🔄 **Workflow Versions**

### **Current Version: v1.0**
- Multi-agent pipeline with quality validation
- Intelligent retry logic with enhanced prompts
- MCP integration for robust web scraping
- Dynamic JSON schema mapping

### **Future Enhancements**
- Batch URL processing capability
- Custom schema templates for different industries
- Advanced analytics and reporting
- Integration with external data sources

## 📝 **Development Notes**

### **Code Nodes**
- **Validate URL**: Basic regex validation with retry counter
- **Code**: JSON parser with markdown cleaning and error handling
- **Parser validation code**: 3-tier extraction with regex fallback
- **Gate quality**: Decision logic for retry vs accept
- **Parser**: Final data cleaner
- **LayoutData**: Template mapping for UI display

### **Webhook Configuration**
```bash
# Chat trigger webhook ID
webhookId: "69b625fe-dd04-4113-bd3a-0b6c7266f670"

# Access URL
https://your-n8n-domain.com/webhook/69b625fe-dd04-4113-bd3a-0b6c7266f670
```

---

## 🤝 **Contributing**

To modify or enhance the workflow:

1. **Backup Current Version**: Export existing workflow before changes
2. **Test Changes**: Use development environment for testing
3. **Document Updates**: Update this README with any changes
4. **Version Control**: Increment version number in workflow name

## 📄 **License**

This workflow is part of the AI Web Extractor project and follows the same MIT License.

---

<div align="center">

**🚀 Ready to extract intelligence from any webpage!**

[⬆️ Back to Main Project](../README.md) • [📖 Full Documentation](../docs/) • [🔧 MCP Server](../mcp-server/)

</div>

## 🔧 **Workflow Configuration**

### **AI Models Used**
- **Planner Agent**: OpenAI GPT-4 Turbo Preview
- **Extractor Agent**: OpenAI GPT-4 Turbo Preview  
- **Validator Agent**: OpenAI GPT-4

### **Key Parameters**
```javascript
// Quality Gate Thresholds
MIN_COMPLETENESS = 0.4    // 40% field completion required
MIN_CONFIDENCE = 0.3      // 30% confidence score required
MAX_RETRIES = 2          // Maximum retry attempts

// URL Validation
URL_TIMEOUT = 5000       // 5 second timeout for accessibility check
RETRY_WAIT = 1000        // 1 second between retries
```

### **JSON Schema Structure**
```json
{
  "passport": {
    "id": "URL or unique identifier",
    "issuedOn": "ISO timestamp",
    "authority": "Domain or company name",
    "eventsCount": "Number of steps",
    "contributorsCount": "Number of actors"
  },
  "steps": [
    {
      "stepNumber": "Sequential number",
      "narrative": {
        "title": "Product/page title",
        "paragraph": "Description content"
      },
      "extraNote": "Price or additional info"
    }
  ],
  "actors": [
    {
      "fullName": "Company/vendor name",
      "country": "Derived from domain",
      "role": "vendor/manufacturer",
      "website": "Company website",
      "sns": "Social media links"
    }
  ]
}
```
