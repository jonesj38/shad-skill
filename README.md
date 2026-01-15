# Shad Skill for Claude Code

A Claude Code skill that enables intelligent use of [Shad](https://github.com/jonesj38/shad) - a recursive reasoning system with Obsidian vault context.

## What is Shad?

Shad enables AI to utilize virtually unlimited context by loading Obsidian vaults with curated knowledge. It recursively decomposes complex tasks, retrieves targeted context for each subtask, generates code with type consistency, and assembles coherent results.

## Installation

### Personal Installation (Available Everywhere)

```bash
# Clone this repo
git clone https://github.com/jonesj38/shad_skill.git

# Copy to Claude Code skills directory
mkdir -p ~/.claude/skills
cp -r shad_skill ~/.claude/skills/shad
```

### Project Installation (Shared via Git)

```bash
# In your project directory
mkdir -p .claude/skills
cp -r /path/to/shad_skill .claude/skills/shad

# Commit to share with your team
git add .claude/skills/shad
git commit -m "Add Shad skill"
```

## Usage

The skill automatically activates when you mention:
- Building production apps
- Complex task decomposition
- Vault context or knowledge bases
- Long-context reasoning

Or invoke directly:
```
/shad
```

### Example Prompts

```
"Help me use Shad to build a task management app using my MobileDevVault"

"How do I set up automated vault ingestion with Shad?"

"Use Shad to analyze authentication patterns in my SecurityDocs vault"
```

## Skill Contents

| File | Description |
|------|-------------|
| `SKILL.md` | Main skill definition with commands and examples |
| `strategies.md` | Detailed strategy documentation (software, research, analysis) |
| `code-mode.md` | Code Mode retrieval patterns and examples |
| `api-reference.md` | REST API endpoints reference |

## Prerequisites

- [Shad](https://github.com/jonesj38/shad) installed and configured
- [Claude Code](https://claude.ai/code) CLI
- An Obsidian vault with relevant content

## Quick Shad Commands

```bash
# Run a task with vault context
shad run "Your task" --vault ~/YourVault

# Build software with verification
shad run "Build API" --vault ~/Docs --strategy software --verify strict --write-files

# Add automated sources
shad sources add github https://github.com/org/repo --schedule weekly --vault ~/Vault

# Check source sync status
shad sources status
```

## Links

- [Shad Repository](https://github.com/jonesj38/shad)
- [Shad Documentation](https://github.com/jonesj38/shad/blob/main/SPEC.md)
- [Claude Code Skills Documentation](https://docs.anthropic.com/claude-code/skills)

## License

MIT
