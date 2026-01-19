# Code Mode

Code Mode is Shad's key innovation for intelligent retrieval. Instead of simple keyword search, the LLM writes Python scripts to retrieve exactly what it needs from your vault.

## How It Works

For each subtask, Shad:
1. Analyzes what context is needed
2. Generates a Python retrieval script
3. Executes the script in a sandbox
4. Uses the retrieved context for generation

## Example Retrieval Script

For task: "How should I implement OAuth?"

```python
# LLM generates this script automatically
results = obsidian.search("OAuth implementation", limit=10)
patterns = obsidian.read_note("Patterns/Authentication/OAuth.md")

relevant = []
for r in results:
    if "refresh token" in r["content"].lower():
        relevant.append(r["content"][:2000])

__result__ = {
    "context": f"## OAuth Patterns\n{patterns[:3000]}\n\n## Examples\n{'---'.join(relevant)}",
    "citations": [{"vault": "main", "path": r["path"], "snip_start": 0, "snip_end": 2000} for r in results[:5]],
    "queries": ["OAuth implementation", "refresh token"],
    "signals": {"num_notes": len(results), "total_chars": sum(len(r["content"]) for r in results), "keyword_hits": 18},
    "confidence": 0.72,
    "why": "Found OAuth pattern note + 3 refresh token examples"
}
```

## Available Tools (ObsidianTools)

Scripts have access to these vault operations via the `obsidian` object:

### search(query, limit=10, path_filter=None, type_filter=None)
Search vault content by keywords.

```python
results = obsidian.search("authentication React Native", limit=10)
# Returns: [{"path": "...", "content": "...", "score": 0.85}, ...]

# With filters
results = obsidian.search("jwt", limit=5, path_filter="Patterns/")
```

### read_note(path)
Read a specific note by path.

```python
content = obsidian.read_note("Patterns/Authentication.md")
# Returns: Full note content as string
```

### list_notes(directory=None, recursive=False)
List notes in vault or folder.

```python
all_notes = obsidian.list_notes()
pattern_notes = obsidian.list_notes("Patterns", recursive=True)
# Returns: ["path1.md", "path2.md", ...]
```

### get_frontmatter(path)
Get note frontmatter/metadata.

```python
meta = obsidian.get_frontmatter("Patterns/Auth.md")
# Returns: {"tags": [...], "created": "...", ...}
```

### get_hash(path)
Get SHA256 hash of note content (for cache validation).

```python
hash_value = obsidian.get_hash("Patterns/Auth.md")
# Returns: "sha256:..."
```

### create_wikilink(path)
Create a wikilink to a note.

```python
link = obsidian.create_wikilink("Patterns/Auth.md")
# Returns: "[[Patterns/Auth]]"
```

### write_note(path, content, note_type=None, frontmatter=None)
Write a note to the vault (HITL gated for safety).

```python
obsidian.write_note(
    "Notes/Summary.md",
    "# Summary\n\nKey findings...",
    frontmatter={"tags": ["summary", "auth"]}
)
```

## Multi-Vault Operations

With layered vaults, you can target specific vaults:

```python
# Search specific vault
results = obsidian.search("jwt", vault="patterns")

# Search multiple vaults
results = obsidian.search("config", vaults=["project", "docs"])
```

## Result Format

Scripts must set `__result__` with:

```python
__result__ = {
    "context": str,        # Retrieved context to use (required)
    "citations": list,     # Source paths for attribution
    "queries": list,       # Queries used for retrieval
    "signals": dict,       # Metrics: num_notes, total_chars, keyword_hits
    "confidence": float,   # 0.0-1.0 retrieval quality
    "why": str            # Explanation of retrieval logic
}
```

Output is capped at 200KB.

## Retrieval Scoring & Recovery

**System-computed relevance score** (independent of script's self-reported confidence):

```
retrieval_score = w1*keyword_overlap + w2*coverage + w3*diversity - w4*boilerplate_penalty
```

**Recovery policy:**

| Tier | Trigger | Action |
|------|---------|--------|
| A | `retrieval_score < 0.45` OR `confidence < 0.5` | Regenerate script with hints (1 retry) |
| B | Tier A failed | Broadened direct search fallback |
| C | Tier B failed AND high-impact node | Human checkpoint |

**Thresholds:**
- `min_context_chars`: 2000
- `min_citations`: 2
- `low_score_threshold`: 0.45
- `max_retrieval_regens_per_node`: 1
- `max_total_retrieval_regens_per_run`: 5

## Multi-Step Retrieval

Scripts can perform complex multi-step retrieval:

```python
# Step 1: Find relevant folders
folders = obsidian.list_notes()
auth_folders = [f for f in folders if "auth" in f.lower()]

# Step 2: Search within relevant folders
all_context = []
for folder in auth_folders:
    results = obsidian.search("OAuth JWT", limit=5)
    all_context.extend([r["content"][:1000] for r in results])

# Step 3: Read specific pattern notes
pattern = obsidian.read_note("Patterns/OAuth.md")
if pattern:
    all_context.append(pattern)

__result__ = {
    "context": "\n---\n".join(all_context),
    "citations": [{"vault": "main", "path": f} for f in auth_folders],
    "confidence": 0.8 if all_context else 0.2,
    "why": f"Found {len(all_context)} relevant sections"
}
```

## Sandbox Security

Scripts run in a sandboxed environment with configurable profiles:

| Profile | File Access | Network | Use Case |
|---------|-------------|---------|----------|
| `strict` (default) | Vault only via ObsidianTools | None | Safe, deterministic |
| `local` | + Read-only to allowlisted roots | None | Reference local repos |
| `extended` | + Read-only allowlist | HTTP GET to allowlisted domains | Fetch live docs |

**Sandbox constraints:**
- **Disabled builtins**: `eval`, `exec`, `compile`, `__import__`, raw `open`
- **Allowed imports**: `json`, `re`, `datetime`, `collections`, `itertools`, `functools`, `math`, `hashlib`, `pathlib`, `typing`, `dataclasses`, `enum`, `yaml`
- **Resource limits**: 60s timeout, 512MB memory
- **Output**: Via `__result__` only, capped at 200KB

**CLI profile usage:**
```bash
shad run "..." --profile strict  # Default
shad run "..." --profile local --fs-read ./myrepo
shad run "..." --profile extended --net-allow docs.example.com
```

## Disabling Code Mode

For simple tasks, disable Code Mode for direct search:

```bash
shad run "Find all notes about React" --vault ~/Vault --no-code-mode
```

This uses simple keyword search instead of LLM-generated scripts.
