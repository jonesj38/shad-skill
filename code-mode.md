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
    "citations": [r["path"] for r in results[:5]],
    "confidence": 0.72,
    "why": "Found OAuth pattern note + 3 refresh token examples"
}
```

## Available Tools (ObsidianTools)

Scripts have access to these vault operations:

### search(query, limit=10)
Search vault content by keywords.

```python
results = obsidian.search("authentication React Native", limit=10)
# Returns: [{"path": "...", "content": "...", "score": 0.85}, ...]
```

### read_note(path)
Read a specific note by path.

```python
content = obsidian.read_note("Patterns/Authentication.md")
# Returns: Full note content as string
```

### list_notes(folder=None)
List notes in vault or folder.

```python
all_notes = obsidian.list_notes()
pattern_notes = obsidian.list_notes("Patterns")
# Returns: ["path1.md", "path2.md", ...]
```

### get_metadata(path)
Get note frontmatter/metadata.

```python
meta = obsidian.get_metadata("Patterns/Auth.md")
# Returns: {"tags": [...], "created": "...", ...}
```

## Result Format

Scripts must set `__result__` with:

```python
__result__ = {
    "context": str,      # Retrieved context to use
    "citations": list,   # Source paths for attribution
    "confidence": float, # 0.0-1.0 retrieval quality
    "why": str          # Explanation of retrieval logic
}
```

## Retrieval Recovery

When retrieval confidence is low, Shad uses tiered recovery:

| Tier | Trigger | Action |
|------|---------|--------|
| A | confidence < 0.5 | Regenerate script with hints |
| B | Tier A fails | Broadened direct search fallback |
| C | High-impact node | Human checkpoint for guidance |

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
if obsidian.read_note("Patterns/OAuth.md"):
    all_context.append(obsidian.read_note("Patterns/OAuth.md"))

__result__ = {
    "context": "\n---\n".join(all_context),
    "citations": auth_folders,
    "confidence": 0.8 if all_context else 0.2,
    "why": f"Found {len(all_context)} relevant sections"
}
```

## Sandbox Security

Scripts run in a sandboxed environment with configurable profiles:

| Profile | File Access | Network | Use Case |
|---------|-------------|---------|----------|
| `strict` (default) | Vault only | None | Safe, deterministic |
| `local` | + Allowlisted roots | None | Reference local repos |
| `extended` | + Allowlist | HTTP GET to allowlist | Fetch live docs |

## Disabling Code Mode

For simple tasks, disable Code Mode for direct search:

```bash
shad run "Find all notes about React" --vault ~/Vault --no-code-mode
```

This uses simple keyword search instead of LLM-generated scripts.
