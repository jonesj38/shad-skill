# Shad API Reference

Shad provides a REST API for programmatic access.

## Base URL

```
http://localhost:8000/v1
```

## Endpoints

### Execute Task

**POST /v1/run**

Execute a reasoning task with vault context.

```bash
curl -X POST http://localhost:8000/v1/run \
  -H "Content-Type: application/json" \
  -d '{
    "task": "Build a login form",
    "vaults": ["/home/user/DevVault"],
    "strategy": "software",
    "max_depth": 3,
    "verify": "basic"
  }'
```

**Request Body:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `task` | string | Yes | Task description |
| `vaults` | string[] | Yes | Vault paths (priority order) |
| `strategy` | string | No | Force strategy |
| `max_depth` | int | No | Max recursion depth (default: 3) |
| `max_nodes` | int | No | Max DAG nodes (default: 50) |
| `max_time` | int | No | Max wall time seconds (default: 300) |
| `verify` | string | No | Verification level |
| `code_mode` | bool | No | Enable Code Mode (default: true) |

**Response:**
```json
{
  "run_id": "abc123",
  "status": "running"
}
```

---

### Get Run Status

**GET /v1/run/:id**

Get status and results of a run.

```bash
curl http://localhost:8000/v1/run/abc123
```

**Response:**
```json
{
  "run_id": "abc123",
  "status": "complete",
  "task": "Build a login form",
  "strategy": "software",
  "result": "...",
  "files": [...],
  "metrics": {
    "nodes_executed": 12,
    "tokens_used": 45000,
    "duration_seconds": 120
  }
}
```

**Status Values:**
- `pending` - Not started
- `running` - In progress
- `complete` - Finished successfully
- `partial` - Partially complete (resumable)
- `failed` - Failed with error
- `aborted` - Manually stopped

---

### Resume Run

**POST /v1/run/:id/resume**

Resume a partial or failed run.

```bash
curl -X POST http://localhost:8000/v1/run/abc123/resume \
  -H "Content-Type: application/json" \
  -d '{
    "replay": "stale"
  }'
```

**Request Body:**
| Field | Type | Description |
|-------|------|-------------|
| `replay` | string | Nodes to replay: `stale`, `all`, or specific node ID |

---

### List Runs

**GET /v1/runs**

List recent runs.

```bash
curl http://localhost:8000/v1/runs?limit=10
```

**Query Parameters:**
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `limit` | int | 20 | Max results |
| `status` | string | - | Filter by status |

---

### Vault Status

**GET /v1/vault/status**

Check vault connection status.

```bash
curl http://localhost:8000/v1/vault/status
```

**Response:**
```json
{
  "connected": true,
  "vault_path": "/home/user/DevVault",
  "note_count": 1234
}
```

---

### Search Vault

**GET /v1/vault/search**

Search vault content directly.

```bash
curl "http://localhost:8000/v1/vault/search?q=authentication&limit=10"
```

**Query Parameters:**
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `q` | string | - | Search query (required) |
| `limit` | int | 10 | Max results |

**Response:**
```json
{
  "results": [
    {
      "path": "Patterns/Auth.md",
      "content": "...",
      "score": 0.92
    }
  ]
}
```

---

## Error Responses

All endpoints return errors in this format:

```json
{
  "error": {
    "code": "INVALID_VAULT",
    "message": "Vault path does not exist",
    "details": {}
  }
}
```

**Common Error Codes:**
| Code | HTTP Status | Description |
|------|-------------|-------------|
| `INVALID_VAULT` | 400 | Vault path invalid |
| `RUN_NOT_FOUND` | 404 | Run ID doesn't exist |
| `BUDGET_EXHAUSTED` | 429 | Token/time budget exceeded |
| `VAULT_UNREACHABLE` | 503 | Cannot connect to vault |

---

## WebSocket Events

For real-time updates, connect to:

```
ws://localhost:8000/v1/run/:id/stream
```

**Event Types:**
```json
{"type": "node_started", "node_id": "...", "task": "..."}
{"type": "node_completed", "node_id": "...", "result": "..."}
{"type": "retrieval", "node_id": "...", "confidence": 0.8}
{"type": "verification", "check": "syntax", "passed": true}
{"type": "run_complete", "status": "complete"}
```
