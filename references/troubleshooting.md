# Troubleshooting Guide

Common issues and solutions when using HAPI CLI for API-to-MCP conversion.

## Installation Issues

### Command not found: hapi

**Cause**: HAPI not in PATH or not installed.

**Solutions**:
```bash
# Check if installed
which hapi

# Re-install
curl -fsSL https://get.mcp.com.ai/hapi.sh | bash

# Or use via Docker
docker run hapimcp/hapi-cli:latest --version
```

### Permission denied during installation

**Cause**: Script trying to write to protected directory.

**Solution**:
```bash
# Install to user directory
curl -fsSL https://get.mcp.com.ai/hapi.sh | HAPI_INSTALL_DIR=~/.local/bin bash

# Ensure directory is in PATH
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

## OpenAPI Spec Issues

### "Invalid OpenAPI specification" error

**Cause**: Spec is v2 (Swagger) or malformed.

**Solutions**:
1. Verify spec version:
   ```bash
   curl -s https://api.example.com/openapi.json | jq '.openapi // .swagger'
   ```
2. If v2, convert to v3 using [Swagger Converter](https://converter.swagger.io/) or via API:
  ```bash
  # Convert using Swagger Converter API (replace URL with your spec)
  curl -s "https://converter.swagger.io/api/convert?url=https://api.example.com/swagger.json" -o openapi.json
  ```

### "Failed to load OpenAPI spec from URL"

**Cause**: Network issue or CORS blocking.

**Solutions**:
```bash
# Test URL accessibility
curl -I https://api.example.com/openapi.json

# Download locally and use local file
curl -o ./openapi.json https://api.example.com/openapi.json
hapi serve myapi --openapi ./openapi.json --url https://api.example.com --headless
```

### Spec not auto-detected

**Cause**: File not in expected location.

**Solution**: Use explicit `--openapi` flag:
```bash
hapi serve myapi --openapi ./path/to/spec.yaml --headless
```

Or place in standard location:
```bash
mkdir -p ~/.hapi/specs
cp ./spec.yaml ~/.hapi/specs/myapi.yaml
hapi serve myapi --headless
```

## Server Startup Issues

### Port already in use

**Cause**: Another process using the port.

**Solutions**:
```bash
# Find process using port
lsof -i :3030

# Use different port
hapi serve myapi --port 3031 --headless

# Or kill existing process
kill $(lsof -t -i :3030)
```

### "ECONNREFUSED" when connecting to backend

**Cause**: Backend URL unreachable.

**Solutions**:
1. Verify backend is running:
   ```bash
   curl -I https://api.example.com/health
   ```
2. Check `--url` flag points to correct backend
3. Ensure no firewall blocking outbound connections

## Deployment Issues

### Cloudflare login fails

**Cause**: Browser not opening or OAuth timeout.

**Solutions**:
1. Ensure default browser is set
2. Try manual login at https://dash.cloudflare.com/
3. Check for Cloudflare API token in environment:
   ```bash
   export CLOUDFLARE_API_TOKEN=your-token
   hapi deploy ...
   ```

### Worker deployment fails

**Cause**: Name conflict, quota exceeded, or auth issues.

**Solutions**:
```bash
# Use unique name
hapi deploy --name my-api-mcp-v2 ...

# Check worker status
hapi status -n my-api-mcp-v2

# Dry run to see what would be deployed
hapi deploy --dry-run ...
```

### Docker container exits immediately

**Cause**: Missing required flags or spec not found.

**Solutions**:
```bash
# Check logs
docker logs hapi-myapi

# Run interactively to debug
docker run -it --rm hapimcp/hapi-cli:latest serve myapi --openapi /app/specs/api.yaml --headless

# Verify volume mounts
docker run -it --rm -v $(pwd)/specs:/app/specs hapimcp/hapi-cli:latest ls /app/specs
```

## MCP Connection Issues

### "/mcp/ping" returns 404

**Cause**: MCP mode not enabled or wrong endpoint.

**Solutions**:
```bash
# Ensure MCP is enabled (default is true)
hapi serve myapi --mcp --headless

# Verify endpoint
curl -s http://localhost:3030/mcp/ping
```

### MCP client can't connect

**Cause**: Wrong transport configuration.

**Solution**: HAPI uses Streamable HTTP, not stdio:
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

### Tool calls return errors

**Cause**: Backend API rejecting requests or auth required.

**Solutions**:
1. Test backend directly:
   ```bash
   curl -X GET https://api.example.com/endpoint
   ```
2. Check if auth headers needed — configure in OpenAPI spec's security schemes
3. Review HAPI logs:
   ```bash
   hapi serve myapi --dev --headless  # Verbose logging
   ```

## Performance Issues

### Slow response times

**Cause**: Backend latency or cold starts (Cloudflare).

**Solutions**:
1. For Cloudflare: First request after idle may be slow (cold start)
2. Check backend performance:
   ```bash
   time curl -s https://api.example.com/endpoint
   ```
3. Use `--dev` to see timing logs

### High memory usage (Docker)

**Cause**: Large OpenAPI spec or many concurrent connections.

**Solution**: Set memory limits:
```bash
docker run --memory=256m --name hapi-myapi ...
```

## Getting Help

If issues persist:

1. Check HAPI documentation: https://hapi.mcp.com.ai
2. Review GitHub issues: https://github.com/la-rebelion/hapimcp/issues
3. Run with verbose logging: `hapi serve ... --dev`
4. Check server logs in `~/.hapi/logs/`
