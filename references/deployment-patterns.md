# Deployment Patterns

Recipes for deploying HAPI MCP servers in various environments.

## Cloudflare Workers (Recommended)

Best for: Public APIs, low latency, zero infrastructure management.

### Basic Deployment

```bash
# 1. Authenticate (first time only)
hapi login

# 2. Deploy
hapi deploy \
  --openapi https://api.example.com/openapi.json \
  --url https://api.example.com \
  --name my-api-mcp

# 3. Verify
curl -s https://my-api-mcp.<account>.workers.dev/health
curl -s https://my-api-mcp.<account>.workers.dev/mcp/ping
```

### With Environment Variables

```bash
hapi deploy \
  --openapi ./specs/api.yaml \
  --url https://api.example.com \
  --name my-api-mcp \
  --var API_KEY=your-api-key \
  --var ENVIRONMENT=production
```

### Custom Domain

After deployment, configure custom domain in Cloudflare dashboard:
1. Go to Workers & Pages → your worker
2. Settings → Triggers → Custom Domains
3. Add your domain (e.g., `mcp.yourdomain.com`)

## Docker

Best for: Self-hosted, private networks, custom infrastructure.

### Basic Container

```bash
docker run --name hapi-myapi -d --rm \
  -p 3030:3030 \
  hapimcp/hapi-cli:latest serve myapi \
  --openapi https://api.example.com/openapi.json \
  --url https://api.example.com \
  --port 3030 \
  --headless
```

### With Local Spec File

```bash
docker run --name hapi-myapi -d --rm \
  -p 3030:3030 \
  -v $(pwd)/specs:/app/specs:ro \
  hapimcp/hapi-cli:latest serve myapi \
  --openapi /app/specs/openapi.yaml \
  --url https://api.example.com \
  --port 3030 \
  --headless
```

### With Persistent Configuration

```bash
docker run --name hapi-myapi -d --rm \
  -p 3030:3030 \
  -v ~/.hapi:/app/.hapi \
  hapimcp/hapi-cli:latest serve myapi \
  --port 3030 \
  --headless
```

### Docker Compose

```yaml
version: '3.8'
services:
  hapi-mcp:
    image: hapimcp/hapi-cli:latest
    container_name: hapi-myapi
    command: serve myapi --openapi /app/specs/openapi.yaml --url https://api.example.com --port 3030 --headless
    ports:
      - "3030:3030"
    volumes:
      - ./specs:/app/specs:ro
      - hapi-config:/app/.hapi
    environment:
      - HAPI_LOG_LEVEL_PROD=info
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3030/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 10s
    restart: unless-stopped

volumes:
  hapi-config:
```

### With Reverse Proxy (Caddy)

```
# Caddyfile
mcp.yourdomain.com {
    reverse_proxy hapi-mcp:3030
}
```

## Local Development

### Quick Start

```bash
# Serve petstore demo
hapi serve petstore \
  --openapi https://petstore3.swagger.io/api/v3/openapi.json \
  --url https://petstore3.swagger.io/api/v3 \
  --headless \
  --dev \
  --port 3030
```

### With TLS (HTTPS)

```bash
hapi serve myapi \
  --openapi ./specs/api.yaml \
  --url https://api.example.com \
  --headless \
  --cert ./certs/localhost.pem \
  --key ./certs/localhost-key.pem \
  --port 3443
```

### Development Mode

Use `--dev` for verbose logging:

```bash
hapi serve myapi --openapi ./specs/api.yaml --headless --dev
```

## Health Checks

All deployments should verify these endpoints:

```bash
# Health endpoint (returns 200 OK)
curl -s http://localhost:3030/health

# MCP ping (returns "pong")
curl -s http://localhost:3030/mcp/ping

# MCP server info (returns 204)
curl -s -I http://localhost:3030/mcp
```

## MCP Client Connection

Configure MCP clients to connect via Streamable HTTP:

```json
{
  "mcpServers": {
    "my-api": {
      "transport": {
        "type": "http",
        "url": "http://localhost:3030/mcp"
      }
    }
  }
}
```

For Cloudflare deployments:

```json
{
  "mcpServers": {
    "my-api": {
      "transport": {
        "type": "http",
        "url": "https://my-api-mcp.<account>.workers.dev/mcp"
      }
    }
  }
}
```
