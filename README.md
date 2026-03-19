# MCP ShortURL

[![PyPI version](https://img.shields.io/pypi/v/mcp-shorturl.svg)](https://pypi.org/project/mcp-shorturl/)
[![PyPI downloads](https://img.shields.io/pypi/dm/mcp-shorturl.svg)](https://pypi.org/project/mcp-shorturl/)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![MCP](https://img.shields.io/badge/MCP-Compatible-green.svg)](https://modelcontextprotocol.io)

A [Model Context Protocol (MCP)](https://modelcontextprotocol.io) server for URL shortening using [Short URL API](https://platform.acedata.cloud/documents/a2303356-6672-4eb8-9778-75f55c998fe9) through the [AceDataCloud API](https://platform.acedata.cloud).

Create short, shareable URLs directly from Claude, VS Code, or any MCP-compatible client.

## Features

- **URL Shortening** - Convert long URLs into short, shareable links
- **Batch Shortening** - Shorten multiple URLs at once (up to 10 per batch)
- **Free Service** - Zero credit consumption per request
- **Permanent Links** - Short URLs never expire
- **surl.id Domain** - Short URLs use the clean `surl.id` domain
- **Bearer Auth** - Secure API access with token authentication

## Quick Start

### 1. Get API Token

Get your API token from [AceDataCloud Platform](https://platform.acedata.cloud):

1. Sign up or log in
2. Navigate to [Short URL API](https://platform.acedata.cloud/documents/a2303356-6672-4eb8-9778-75f55c998fe9)
3. Click "Acquire" to get your token

### 2. Install

```bash
# Clone the repository
git clone https://github.com/AceDataCloud/mcp-shorturl.git
cd mcp-shorturl

# Install with pip
pip install -e .

# Or with uv (recommended)
uv pip install -e .
```

### 3. Configure

```bash
# Copy example environment file
cp .env.example .env

# Edit with your API token
echo "ACEDATACLOUD_API_TOKEN=your_token_here" > .env
```

### 4. Run

```bash
# Run the server
mcp-shorturl

# Or with Python directly
python main.py
```

## Claude Desktop Integration

Add to your Claude Desktop configuration:

**macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
**Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "shorturl": {
      "command": "mcp-shorturl",
      "env": {
        "ACEDATACLOUD_API_TOKEN": "your_api_token_here"
      }
    }
  }
}
```

Or if using uv:

```json
{
  "mcpServers": {
    "shorturl": {
      "command": "uv",
      "args": ["run", "--directory", "/path/to/mcp-shorturl", "mcp-shorturl"],
      "env": {
        "ACEDATACLOUD_API_TOKEN": "your_api_token_here"
      }
    }
  }
}
```

## Remote HTTP Mode (Hosted)

AceDataCloud hosts a managed MCP server that you can connect to directly — **no local installation required**.

**Endpoint**: `https://shorturl.mcp.acedata.cloud/mcp`

All requests require a Bearer token in the `Authorization` header. Get your token from [AceDataCloud Platform](https://platform.acedata.cloud).

### Claude Desktop (Remote)

```json
{
  "mcpServers": {
    "shorturl": {
      "type": "streamable-http",
      "url": "https://shorturl.mcp.acedata.cloud/mcp",
      "headers": {
        "Authorization": "Bearer your_api_token_here"
      }
    }
  }
}
```

### Cursor / VS Code

In your MCP client settings, add:

- **Type**: `streamable-http`
- **URL**: `https://shorturl.mcp.acedata.cloud/mcp`
- **Headers**: `Authorization: Bearer your_api_token_here`

### JetBrains IDEs

Install the [ShortURL MCP plugin](https://plugins.jetbrains.com/plugin/com.acedatacloud.mcp.shorturl) from the JetBrains Marketplace, or configure manually:

1. Go to **Settings → Tools → AI Assistant → Model Context Protocol (MCP)**
2. Click **Add** and select **HTTP**
3. Paste this configuration:

```json
{
  "mcpServers": {
    "shorturl": {
      "url": "https://shorturl.mcp.acedata.cloud/mcp",
      "headers": {
        "Authorization": "Bearer your_api_token_here"
      }
    }
  }
}
```

### cURL Test

```bash
# Health check (no auth required)
curl https://shorturl.mcp.acedata.cloud/health

# MCP initialize (requires Bearer token)
curl -X POST https://shorturl.mcp.acedata.cloud/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer your_api_token_here" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"test","version":"1.0"}}}'
```

### Self-Hosting with Docker

```bash
docker pull ghcr.io/acedatacloud/mcp-shorturl:latest
docker run -p 8000:8000 ghcr.io/acedatacloud/mcp-shorturl:latest
```

Clients connect with their own Bearer token — the server extracts the token from each request's `Authorization` header and uses it for upstream API calls.

## Available Tools

### URL Shortening Tools

| Tool                    | Description                            |
| ----------------------- | -------------------------------------- |
| `shorturl_create`       | Shorten a single URL                   |
| `shorturl_batch_create` | Shorten multiple URLs at once (max 10) |

### Information Tools

| Tool                       | Description                     |
| -------------------------- | ------------------------------- |
| `shorturl_get_usage_guide` | Get comprehensive usage guide   |
| `shorturl_get_api_info`    | Get API details and error codes |

## Usage Examples

### Shorten a Single URL

```
User: Shorten this URL: https://platform.acedata.cloud/documents/a2303356-6672-4eb8-9778-75f55c998fe9

Claude: I'll shorten that URL for you.
[Calls shorturl_create with url="https://platform.acedata.cloud/documents/a2303356-6672-4eb8-9778-75f55c998fe9"]

Result: https://surl.id/1uHCs01xa5
```

### Batch Shorten Multiple URLs

```
User: Shorten these URLs for my social media posts:
- https://example.com/blog/very-long-article-title-about-ai
- https://example.com/products/new-release-2024

Claude: I'll shorten both URLs at once.
[Calls shorturl_batch_create with urls=[...]]
```

### Create Links for Documentation

```
User: I need clean short links for these reference URLs in my doc.

Claude: I'll create short links for all your references.
[Calls shorturl_batch_create with the list of URLs]
```

## Response Structure

### Successful Response

```json
{
  "success": true,
  "data": {
    "url": "https://surl.id/1uHCs01xa5"
  }
}
```

### Error Response

```json
{
  "success": false,
  "error": {
    "code": "api_error",
    "message": "fetch failed"
  },
  "trace_id": "2cf86e86-22a4-46e1-ac2f-032c0f2a4e89"
}
```

## Configuration

### Environment Variables

| Variable                    | Description                 | Default                     |
| --------------------------- | --------------------------- | --------------------------- |
| `ACEDATACLOUD_API_TOKEN`    | API token from AceDataCloud | **Required**                |
| `ACEDATACLOUD_API_BASE_URL` | API base URL                | `https://api.acedata.cloud` |
| `SHORTURL_REQUEST_TIMEOUT`  | Request timeout in seconds  | `30`                        |
| `LOG_LEVEL`                 | Logging level               | `INFO`                      |

### Command Line Options

```bash
mcp-shorturl --help

Options:
  --version          Show version
  --transport        Transport mode: stdio (default) or http
  --port             Port for HTTP transport (default: 8000)
```

## Development

### Setup Development Environment

```bash
# Clone repository
git clone https://github.com/AceDataCloud/mcp-shorturl.git
cd mcp-shorturl

# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # or `.venv\Scripts\activate` on Windows

# Install with dev dependencies
pip install -e ".[dev,test]"
```

### Run Tests

```bash
# Run unit tests
pytest

# Run with coverage
pytest --cov=core --cov=tools

# Run integration tests (requires API token)
pytest tests/test_integration.py -m integration
```

### Code Quality

```bash
# Format code
ruff format .

# Lint code
ruff check .

# Type check
mypy core tools
```

### Build & Publish

```bash
# Install build dependencies
pip install -e ".[release]"

# Build package
python -m build

# Upload to PyPI
twine upload dist/*
```

## Project Structure

```
MCPShortURL/
├── core/                   # Core modules
│   ├── __init__.py
│   ├── client.py          # HTTP client for ShortURL API
│   ├── config.py          # Configuration management
│   ├── exceptions.py      # Custom exceptions
│   └── server.py          # MCP server initialization
├── tools/                  # MCP tool definitions
│   ├── __init__.py
│   ├── shorturl_tools.py  # URL shortening tools
│   └── info_tools.py      # Information tools
├── prompts/                # MCP prompt templates
│   └── __init__.py
├── tests/                  # Test suite
│   ├── conftest.py
│   ├── test_client.py
│   ├── test_config.py
│   └── test_integration.py
├── deploy/                 # Deployment configs
│   ├── run.sh
│   └── production/
│       ├── deployment.yaml
│       ├── ingress.yaml
│       └── service.yaml
├── .env.example           # Environment template
├── .gitignore
├── .ruff.toml             # Ruff linter configuration
├── CHANGELOG.md
├── Dockerfile             # Docker image for HTTP mode
├── docker-compose.yaml    # Docker Compose config
├── LICENSE
├── main.py                # Entry point
├── pyproject.toml         # Project configuration
└── README.md
```

## API Reference

This server wraps the [AceDataCloud Short URL API](https://platform.acedata.cloud/documents/a2303356-6672-4eb8-9778-75f55c998fe9):

- **Endpoint**: `POST /shorturl`
- **Input**: `{ "content": "https://long-url.example.com/..." }`
- **Output**: `{ "success": true, "data": { "url": "https://surl.id/..." } }`
- **Pricing**: Free (0 credits)
- **Auth**: Bearer token

Full API documentation: [AceDataCloud Platform](https://platform.acedata.cloud/documents/a2303356-6672-4eb8-9778-75f55c998fe9)

## License

MIT License - see [LICENSE](LICENSE) for details.
