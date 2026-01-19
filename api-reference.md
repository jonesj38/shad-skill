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
    "goal": "Build a REST API for user management",
    "vaults": [
      {"id": "project", "root": "/vaults/proj", "priority": 0},
      {"id": "patterns", "root": "/vaults/patterns", "priority": 1}
    ],
    "strategy": "software",
    "config": {
      "max_depth": 4,
      "max_tokens": 100000,
      "verify": "basic"
    },
    "write_files": false,
    "profile": "strict"
  }'
```

**Request Body:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `goal` | string | Yes | Task description |
| `vaults` | object[] | Yes | Vault configs with id, root, priority |
| `strategy` | string | No | Force strategy (software\|research\|analysis\|planning) |
| `config.max_depth` | int | No | Max recursion depth (default: 3) |
| `config.max_nodes` | int | No | Max DAG nodes (default: 50) |
| `config.max_tokens` | int | No | Max token budget (default: 100000) |
| `config.verify` | string | No | Verification level (off\|basic\|build\|strict) |
| `write_files` | bool | No | Write output files to disk (default: false) |
| `profile` | string | No | Sandbox profile (strict\|local\|extended) |

**Response:**
```json
{
  "run_id": "abc123",
  "status": "RUNNING"
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
  "status": "SUCCESS",
  "result": "...",
  "manifest": {
    "files": [
      {
        "path": "src/types.ts",
        "content": "export interface User { ... }",
        "language": "ts",
        "mode": "create",
        "hash": "sha256:...",
        "source_nodes": ["types_contracts"]
      }
    ],
    "notes": [
      {"kind": "contract_change_request", "detail": "Add UserRole enum"}
    ]
  },
  "metrics": {
    "total_tokens": 45000,
    "duration_ms": 120000,
    "nodes_executed": 23
  }
}
```

**Run States:**
| State | Description |
|-------|-------------|
| `PENDING` | Not started |
| `RUNNING` | In progress |
| `SUCCESS` | Finished successfully, meets acceptance criteria |
| `PARTIAL` | Produced artifacts but did not meet criteria |
| `FAILED` | Could not produce meaningful artifacts or safety stop |
| `NEEDS_HUMAN` | Paused, awaiting human input |

---

### Resume Run

**POST /v1/run/:id/resume**

Resume a partial or paused run.

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
| `replay` | string | Nodes to replay: `stale`, `all`, `subtree:<node_id>`, or specific node ID |

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

### Health Check

**GET /v1/health**

Check API health status.

```bash
curl http://localhost:8000/v1/health
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
{"type": "run_complete", "status": "SUCCESS"}
```

---

## DAG Node States

Nodes in the execution DAG can have the following states:

| State | Description |
|-------|-------------|
| `CREATED` | Node exists but not started |
| `STARTED` | Execution in progress |
| `SUCCEEDED` | Completed successfully |
| `CACHE_HIT` | Result retrieved from cache |
| `FAILED` | Execution failed |
| `PRUNED` | Skipped (novelty check or budget) |
