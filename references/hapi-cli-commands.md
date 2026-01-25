# HAPI CLI Commands Reference

Complete reference for HAPI MCP CLI commands and options.

## Installation

```bash
# Linux/macOS
curl -fsSL https://get.mcp.com.ai/hapi.sh | bash

# Windows PowerShell
irm https://get.mcp.com.ai/hapi.ps1 | iex

# Docker
docker run hapimcp/hapi-cli:latest <command>
```

## Commands

### `hapi serve <project> [options]`

Start a local MCP server from an OpenAPI specification.

| Option | Description | Default |
|--------|-------------|---------|
| `--port <number>` | Server port | `3000` |
| `--headless` | Act as headless API proxy (no UI) | `false` |
| `--mcp` | Enable MCP mode | `true` |
| `--dev` | Development mode with verbose logging | `false` |
| `--openapi <pathOrUrl>` | OpenAPI spec file or URL | Auto-detect |
| `--url <backendUrl>` | Backend API URL for proxying | — |
| `--filename <files>` | Comma-separated override files | — |
| `--cors <origins>` | Comma-separated CORS origins | `*` |
| `--cert <path>` | TLS certificate file | — |
| `--key <path>` | TLS private key file | — |

**Examples:**
```bash
# Serve with auto-detected spec
hapi serve myapi --headless --port 3030

# Serve with explicit spec URL
hapi serve myapi --openapi https://api.example.com/openapi.json --url https://api.example.com --headless

# Development mode with TLS
hapi serve myapi --dev --cert ./cert.pem --key ./key.pem --port 3443
```

### `hapi deploy [options]`

Deploy MCP server to Cloudflare Workers.

| Option | Description | Required |
|--------|-------------|----------|
| `--openapi <pathOrUrl>` | OpenAPI spec URL or path | Yes |
| `--name <workerName>` | Cloudflare Worker name | Yes |
| `--url <backendUrl>` | Backend URL for headless proxying | Recommended |
| `--var KEY=VALUE` | Environment variables (repeatable) | No |
| `--dry-run` | Preview deployment without executing | No |

**Examples:**
```bash
# Basic deployment
hapi deploy --openapi https://api.example.com/openapi.json --name my-mcp-worker --url https://api.example.com

# With environment variables
hapi deploy --openapi ./specs/api.yaml --name my-worker --url https://api.example.com --var API_KEY=secret --var ENV=prod

# Dry run to preview
hapi deploy --openapi ./specs/api.yaml --name my-worker --dry-run
```

### `hapi login`

Authenticate with Cloudflare for deployments. Opens browser for OAuth flow.

```bash
hapi login
```

### `hapi list`

List available local API specs in `~/.hapi/specs/`.

```bash
hapi list
```

### `hapi help`

Show help information.

```bash
hapi help
hapi serve --help
hapi deploy --help
```

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `HAPI_HOME` | Config and specs directory | `~/.hapi` |
| `HAPI_PROJECT_NAME` | Project name for spec discovery | — |
| `HAPI_OPENAPI` | OpenAPI spec URL | — |
| `HAPI_URL` | Backend URL for headless proxying | — |
| `HEADLESS` / `HAPI_HEADLESS` | Enable headless mode | `false` |
| `MCP` / `HAPI_MCP` | Enable MCP mode | `true` |
| `MCP_PORT` | Server port | `3000` |
| `HAPI_LOG_LEVEL_DEV` | Dev log level | `debug` |
| `HAPI_LOG_LEVEL_PROD` | Production log level | `info` |

## OpenAPI Spec Resolution (in local serve)

HAPI looks for specs in this order:

1. Exact path provided via `--openapi`
2. `<project>.yaml` in current directory
3. `<project>.json` in current directory
4. `~/.hapi/specs/<project>.yaml`
5. `~/.hapi/specs/<project>.json`
6. `~/.hapi/specs/openapi.yaml`
7. `~/.hapi/specs/openapi.json`

## Override Files

Use `--filename` to merge additional configuration:

```bash
hapi serve myapi --filename security.yaml,paths-override.yaml
```

Override file kinds:
- **Paths**: Additional or modified API paths
- **Schemas**: Additional data schemas
- **Security**: Security scheme overrides
- **ServerConfig**: Feature flags and server settings

Files must have top-level keys matching their kind (e.g., `paths:` for Paths files).