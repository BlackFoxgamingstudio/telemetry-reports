# Architecture: Sovereign Telemetry Reports

## Overview

**Package ID:** `PKG-015`  
**Domain:** Automated Document Compilation  
**Microservice Port:** `8793`  
**n8n Webhook Path:** `telemetry-reports-trigger`  
**GitHub:** [BlackFoxgamingstudio/telemetry-reports](https://github.com/BlackFoxgamingstudio/telemetry-reports)

Automated report compilation engine. Ingests telemetry streams, compiles PDF/DOCX/HTML reports with charts, and distributes via email or webhook on schedule.

---

## System Architecture

```
                     ┌──────────────────────────────────┐
                     │       Sovereign Telemetry Reports   │
                     │       Port: 8793            │
                     ├──────────────┬───────────────────┤
   n8n Webhook ────▶ │  REST API    │   Core Engine     │
   HTTP POST         │  /api/v1/*   │   Dispatcher      │
                     └──────┬───────┴────────┬──────────┘
                            │                │
              ┌─────────────▼────────────────▼─────────┐
              │          Component Layer                 │
              │  TelemetryIngest | ChartRenderer   | PDFCompiler   │
              └────────────────────────┬────────────────┘
                                       │
              ┌────────────────────────▼────────────────┐
              │      n8n Central Event Bus (:5678)       │
              └─────────────────────────────────────────┘
```

## Core Components

### `TelemetryIngester`
Handles all telemetryingester operations. Exposes async methods callable from the core dispatcher.

### `ChartRenderer`
Handles all chartrenderer operations. Exposes async methods callable from the core dispatcher.

### `PDFCompiler`
Handles all pdfcompiler operations. Exposes async methods callable from the core dispatcher.

### `DOCXBuilder`
Handles all docx operations. Exposes async methods callable from the core dispatcher.

### `DistributionManager`
Handles all distribution operations. Exposes async methods callable from the core dispatcher.

---

## API Contract

All interactions follow the SBB standard envelope:

```http
POST /api/v1/execute
Content-Type: application/json
X-SBB-API-Key: <api-key>

{
  "action": "<operation>",
  "payload": {},
  "trace_id": "optional-uuid"
}
```

**Success Response (HTTP 200):**
```json
{
  "status": "success",
  "data": {},
  "trace_id": "...",
  "timestamp": "2025-01-01T00:00:00Z"
}
```

**Health Check:**
```http
GET /health
→ {"status": "healthy", "service": "sovereign-telemetry-reports", "port": 8793}
```

## Integration Matrix

| System | Protocol | Direction | Purpose |
|--------|----------|-----------|---------|
| n8n Event Bus (:5678) | HTTP POST | Outbound | Event forwarding |
| n8n Webhook | HTTP POST | Inbound | Trigger execution |
| SBB Codebase Vault (:8766) | HTTP | Outbound | Code analysis |
| SBB Patterns Bible (:8794) | HTTP | Outbound | Standards validation |
| External APIs | HTTPS | Outbound | Domain-specific data |

## Deployment Architecture

```yaml
# docker-compose excerpt
sovereign-telemetry-reports:
  image: sovereign-telemetry-reports:latest
  ports: ["8793:8793"]
  healthcheck:
    test: curl -f http://localhost:8793/health
    interval: 30s
```

## Security Model

| Control | Implementation |
|---------|---------------|
| Authentication | `X-SBB-API-Key` header (env: `SBB_API_KEY`) |
| Rate Limiting | 100 req/min per client IP |
| Input Validation | Pydantic models (strict mode) |
| Container Security | Non-root user (`appuser:1001`) |
| Secrets | Environment variables only (never hardcoded) |
| TLS | Terminate at reverse proxy (nginx/caddy) |

## Tags
`reporting`, `telemetry`, `pdf`, `charts`
