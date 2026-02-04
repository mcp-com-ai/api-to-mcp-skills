<!--
MCP.com.ai — GitHub Profile README
Source reference: README-mcp-ai.md (MCP.com.ai parent project)
-->

<p align="center">
  <img src="https://img.shields.io/badge/MCP-Model%20Context%20Protocol-1193b0?style=for-the-badge" />
  <img src="https://img.shields.io/badge/API--First%20for%20AI-OpenAPI%20%E2%86%92%20MCP-da7756?style=for-the-badge" />
</p>

<h1 align="center">API-to-MCP (Swagger-to-MCP) Skill</h1>

<p align="center">
  <b>API-First for AI.</b><br/>
  Turn existing <b>OpenAPI v3 specs</b> into <b>MCP servers</b> you can deploy, secure, and operate like real software — with HAPI CLI and zero custom code.
</p>

<p align="center">
  <a href="https://mcp.com.ai"><b>Website</b></a>
  ·
  <a href="https://docs.mcp.com.ai"><b>Docs</b></a>
  ·
  <a href="https://registry.modelcontextprotocol.io/?q=ai.com.mcp"><b>MCP Servers</b></a>  
  ·
  <a href="https://run.mcp.com.ai"><b>Run MCP</b></a>
  ·
  <a href="https://hapi.mcp.com.ai"><b>CLI Tool</b></a>
</p>

---

## 🪝 Why this exists

Most orgs don’t have an “AI problem”.  
They have an **integration and governance** problem.

- Your tools live behind APIs
- Your teams ship OpenAPI specs (or *should*)
- Your agents/IDEs need **safe, auditable access**
- You need **control**: identity, policy, rate limits, logs, approvals, isolation

This skill is a focused bridge for **OpenAPI → MCP** workflows:
- **OpenAPI v3 → MCP** (standard-driven)
- **Deploy anywhere** (local, Docker, Cloudflare Workers)
- **Security-first patterns** (authority outside the model)

---

## ⚡ What you’ll find here

### ✅ Skill-aligned workflow
- Environment setup (HAPI CLI + OpenAPI v3 validation)
- Local serve and MCP endpoint verification
- Deployment options (Cloudflare Workers or Docker)
- Verification checklist for operational readiness

### ✅ Operational patterns
- Safe defaults (timeouts, retries)
- Predictable auth strategies
- Minimal schemas to reduce token bloat

### ✅ References
- CLI commands, deployment patterns, troubleshooting

---

## 🚀 Quickstart (Skill Summary)

### Requirements
- HAPI CLI (`hapi`)
- `curl`
- Optional: `docker` for container deployment
- Optional: `wrangler` for Cloudflare Workers

### Option A — Start a local MCP server
```bash
hapi serve <project-name> \
  --openapi <path-or-url-to-openapi-spec> \
  --url <backend-api-url> \
  --headless \
  --port 3030
```

Verify health and MCP transport:
```bash
curl -s http://localhost:3030/health
curl -s http://localhost:3030/mcp/ping
```

### Option B — Deploy

**Cloudflare Workers**
```bash
hapi login
hapi deploy \
  --openapi <path-or-url-to-openapi-spec> \
  --url <backend-api-url> \
  --name <worker-name> \
  --project <project-name>
```

**Docker (self-hosted)**
```bash
docker run --name hapi-<project> -d --rm \
  -p 3030:3030 \
  hapimcp/hapi-cli:latest serve <project-name> \
  --openapi <openapi-url> \
  --url <backend-api-url> \
  --port 3030 \
  --headless
```

---

## 🔐 Security stance (simple but strict)

**Reasoning stays with the model. Authority stays with the system.**

We optimize for:
- **no long-lived secrets in the model**
- **scoped, short-lived credentials**
- **policy enforcement at the boundary**
- **audit logs by default**
- **transport that fits the environment** (cloud, enterprise, airgap)

---

## 🧪 Testing & eval (MCP quality is measurable)

Good MCP isn’t just “it runs”.
It’s:
- correct tool selection
- stable schemas
- predictable pagination
- safe error behavior
- regression coverage with real prompts

We support evaluation patterns inspired by modern “skills” tooling:
- scenario suites
- tool-call assertions
- latency + cost budgets
- golden traces

---

## 🧭 Who this is for

- **Platform teams** building internal AI enablement
- **DevOps/SRE** who will be on-call for “AI integrations”
- **API teams** who already own OpenAPI contracts
- **Security teams** who need guardrails, identity, and auditability
- **Enterprise architects** tired of one-off agent glue

---

## 🤝 Contribute

We love contributions that make MCP more *operational*:

- new MCP server examples
- OpenAPI → MCP mapping improvements
- eval suites for servers
- security patterns (auth, policy, isolation)
- docs that help non-developers succeed

**How to contribute**
1. Open an issue describing the server or improvement
2. Provide a minimal spec + example prompts
3. Submit PR with tests (when possible)

---

## 📚 References

- [HAPI CLI commands](references/hapi-cli-commands.md)
- [Deployment patterns](references/deployment-patterns.md)
- [Troubleshooting](references/troubleshooting.md)

## 📣 Stay close

If you want MCP to be more than a demo, follow the ecosystem:
- MCP.com.ai updates (docs + servers)
- HAPI MCP Stack releases (deploy, registry, lifecycle)

> Building MCP isn’t the hard part.  
> **Operating MCP safely is the product.**

---

<p align="center">
  <img src="https://img.shields.io/badge/La%20Rebelion%20Labs-Building%20API--First%20for%20AI-0d1117?style=for-the-badge&labelColor=1193b0&color=da7756" />
</p>
