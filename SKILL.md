---
name: shad
description: Recursive reasoning with Obsidian vault context for complex projects. Use when building production apps, decomposing complex tasks, leveraging vault knowledge, or when the user needs long-context reasoning beyond a single context window.
---

# Shad (Shannon's Daemon)

Shad enables AI to utilize virtually unlimited context by loading Obsidian vaults with curated knowledge.

## When to Use This Skill

- Building production-quality applications with specific patterns
- Decomposing complex tasks into manageable subtasks
- Leveraging large documentation sets or knowledge bases
- Generating code that follows specific architectural patterns
- Tasks requiring reasoning over many documents

## Quick Start

```bash
# Basic task with vault context
shad run "Your task" --vault ~/YourVault

# Build software with verification
shad run "Build a REST API" --vault ~/DevDocs --strategy software --verify strict --write-files --output ./api

# Research task
shad run "Summarize key concepts" --vault ~/Research --strategy research
```

## Core Commands

### Running Tasks

```bash
shad run "Task description" --vault /path/to/vault [options]

Options:
  --vault, -v       Path to Obsidian vault (repeatable)
  --strategy        Force strategy: software|research|analysis|planning
  --max-depth, -d   Maximum recursion depth (default: 3)
  --verify          Verification level: off|basic|build|strict
  --write-files     Write output files to disk
  --output, -o      Output directory
```

### Managing Runs

```bash
shad status <run_id>           # Check run status
shad trace tree <run_id>       # View execution DAG
shad resume <run_id>           # Resume partial run
shad export <run_id> -o ./out  # Export files
```

### Vault Ingestion

```bash
# Ingest GitHub repos
shad ingest github https://github.com/org/repo --preset docs --vault ~/Vault

# Presets: mirror (all), docs (documentation), deep (with code)
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

1. **Strategy Selection**: Chooses decomposition approach (software, research, analysis)
2. **Decomposition**: Breaks task into subtasks using strategy skeletons
3. **Code Mode**: LLM generates Python scripts to retrieve vault context
4. **Execution**: Runs retrieval scripts in sandboxed environment
5. **Verification**: Checks syntax, types, imports based on strictness
6. **Synthesis**: Combines subtask results into coherent output

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
| `basic` | Imports + syntax | Default |
| `build` | + Type checking | Pre-commit |
| `strict` | + Tests | Production |

## Vault Preparation Tips

For best results, your vault should include:

**For Software Development:**
- Framework documentation (Markdown)
- Code examples with explanations
- Architecture patterns
- Team coding standards

**For Research:**
- Paper summaries
- Key quotes with citations
- Concept explanations

**General Tips:**
- Use consistent frontmatter
- Keep notes focused (one concept per note)
- Link related notes
- Include worked examples

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
