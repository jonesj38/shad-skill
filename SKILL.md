---
name: shad
description: Recursive reasoning with Obsidian vault context for complex projects. Use when building production apps, decomposing complex tasks, leveraging vault knowledge, or when the user needs long-context reasoning beyond a single context window.
---

# Shad (Shannon's Daemon)

Shad enables AI to utilize virtually unlimited context by treating an Obsidian vault as an explorable environment rather than a fixed input.

## When to Use This Skill

- Building production-quality applications with specific patterns
- Decomposing complex tasks into manageable subtasks
- Leveraging large documentation sets or knowledge bases
- Generating code that follows specific architectural patterns
- Tasks requiring reasoning over many documents

## Core Premise

> **Long-context reasoning is an inference problem, not a prompting problem.**

Instead of cramming context into a single prompt, Shad:
1. **Decomposes** complex tasks into subtasks recursively
2. **Retrieves** targeted context for each subtask via Code Mode
3. **Generates** outputs informed by relevant examples
4. **Assembles** results into coherent output

## Quick Start

```bash
# Basic task with vault context
shad run "Your task" --vault ~/YourVault

# Or set OBSIDIAN_VAULT_PATH and omit --vault
export OBSIDIAN_VAULT_PATH=~/YourVault
shad run "Your task"

# Build software with verification
shad run "Build a REST API" --vault ~/DevDocs --strategy software --verify strict --write-files --output ./api

# Research task
shad run "Summarize key concepts" --vault ~/Research --strategy research

# Multi-vault (priority order)
shad run "Build auth system" --vault ~/Project --vault ~/Patterns --vault ~/Docs
```

## Core Commands

### Running Tasks

```bash
shad run "Task description" --vault /path/to/vault [options]

Options:
  --vault, -v       Path to Obsidian vault (optional if OBSIDIAN_VAULT_PATH set; repeatable)
  --strategy        Force strategy: software|research|analysis|planning
  --max-depth, -d   Maximum recursion depth (default: 3)
  --max-nodes       Maximum DAG nodes (default: 50)
  --max-time, -t    Maximum wall time seconds (default: 300)
  --max-tokens      Maximum token budget (default: 100000)
  --verify          Verification level: off|basic|build|strict
  --write-files     Write output files to disk
  --output, -o      Output directory
  --profile         Sandbox profile: strict|local|extended
  --no-code-mode    Disable Code Mode (direct search only)
  --no-cache        Bypass cache
  --no-checkpoints  Disable non-safety checkpoints
```

### Managing Runs

```bash
shad status <run_id>                  # Check run status
shad trace tree <run_id>              # View execution DAG
shad trace node <run_id> <node_id>    # Inspect specific node
shad resume <run_id>                  # Resume partial run
shad resume <run_id> --replay stale   # Replay stale nodes
shad export <run_id> -o ./out         # Export files
```

### Vault Ingestion

```bash
# Ingest GitHub repos
shad ingest github https://github.com/org/repo --preset docs --vault ~/Vault

# Presets: mirror (all files), docs (documentation), deep (with semantic index)
```

### Sources Scheduler

Automatically sync content on a schedule:

```bash
shad sources add github <url> --schedule weekly --vault ~/Vault
shad sources add url <url> --schedule daily --vault ~/Vault
shad sources add feed <rss_url> --schedule hourly --vault ~/Vault
shad sources add folder <path> --schedule daily --vault ~/Vault

shad sources list              # List all sources
shad sources status            # Detailed sync status
shad sources sync              # Sync due sources
shad sources sync --force      # Force sync all
shad sources remove <id>       # Remove a source
```

Schedules: `manual`, `hourly`, `daily`, `weekly`, `monthly`

### Server Management

```bash
shad server start    # Start Redis + API
shad server stop     # Stop services
shad server status   # Check status
shad server logs -f  # Follow logs
```

## How Shad Works

```
shad run "Build app" --vault ~/MyVault
         |
         v
    RLM Engine
         |
         +-- Strategy Selection -> Decomposition (DAG of subtasks)
         |
         +-- For each node:
         |     Code Mode -> CodeExecutor -> ObsidianTools -> Vault
         |                  (sandboxed)
         |
         +-- Verification Layer (syntax, types, imports)
         |
         +-- Synthesis -> File Manifest
```

## Strategies

| Strategy | Use Case | Key Features |
|----------|----------|--------------|
| `software` | Building applications | Contracts-first, two-pass imports, file manifests |
| `research` | Analysis and summarization | Multi-source synthesis, citation tracking |
| `analysis` | Data analysis | Pattern detection, structured output |
| `planning` | Project planning | Dependency mapping, milestone generation |

## Verification Levels

| Level | Checks | Use When |
|-------|--------|----------|
| `off` | None | Quick iteration |
| `basic` | Imports + syntax + manifest | Default |
| `build` | + Type checking | Pre-commit |
| `strict` | + Tests | Production |

## Sandbox Profiles

| Profile | File Access | Network | Use Case |
|---------|-------------|---------|----------|
| `strict` (default) | Vault only via ObsidianTools | None | Safe, deterministic |
| `local` | + Read-only allowlisted paths | None | Reference local repos |
| `extended` | + Read-only allowlist | HTTP GET to allowlist | Fetch live docs |

## Hard Invariants

1. **Never Auto-Publish**: No irreversible side effects without human approval
2. **Never Exfiltrate**: No sending data externally unless explicitly permitted
3. **Never Self-Modify**: Cannot change own Skills/CORE without human review

## Configuration

Environment variables (or `.env` file):
- `OBSIDIAN_VAULT_PATH`: Default vault path (CLI `--vault` overrides)
- `REDIS_URL`: Redis connection (default: `redis://localhost:6379/0`)

## Vault Preparation Tips

For best results, your vault should include:

**For Software Development:**
- Framework documentation (Markdown)
- Code examples with explanations
- Architecture decision records
- Common patterns and anti-patterns
- Team coding standards

**For Research:**
- Paper summaries
- Key quotes with citations
- Concept explanations
- Related work connections

**General Tips:**
- Use consistent frontmatter for better filtering
- Include code examples with context, not just snippets
- Link related notes for better discovery
- Keep notes focused (one concept per note)

## Example Workflows

### Build a Production App

```bash
# 1. Prepare vault with relevant docs
shad ingest github https://github.com/facebook/react-native --preset docs --vault ~/MobileVault
shad sources add url https://reactnative.dev/docs/getting-started --schedule weekly --vault ~/MobileVault

# 2. Run the build task
shad run "Build a task management app with auth and offline sync" \
  --vault ~/MobileVault \
  --strategy software \
  --verify strict \
  --write-files --output ./TaskApp

# 3. Check results
shad status <run_id>
shad trace tree <run_id>
```

### Research Task

```bash
shad run "Compare authentication approaches in my documentation" \
  --vault ~/SecurityDocs \
  --strategy research \
  --max-depth 2
```

## Additional Resources

For detailed specifications, see:
- [strategies.md](strategies.md) - Strategy skeletons and configuration
- [code-mode.md](code-mode.md) - Code Mode retrieval patterns
- [api-reference.md](api-reference.md) - API endpoints
