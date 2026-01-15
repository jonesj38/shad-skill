# Shad Strategies

Strategies define how Shad decomposes complex tasks into manageable subtasks.

## Software Strategy

For building applications and generating code.

### Required Stages
1. `clarify_requirements` - Understand what needs to be built
2. `project_layout` - Define file structure
3. `types_contracts` - Define interfaces first (contracts-first)
4. `implementation` - Generate code
5. `verification` - Check syntax, types, tests
6. `synthesis` - Assemble final output

### Optional Stages
- `db_schema` - Database design
- `auth` - Authentication setup
- `openapi` - API specification

### Constraints
- `contracts_first: true` - Types must be defined before implementation
- `imports_must_resolve: true` - All imports validated

### Example
```bash
shad run "Build a user management API" \
  --vault ~/APIDocs \
  --strategy software \
  --verify strict
```

---

## Research Strategy

For analysis, summarization, and synthesis tasks.

### Required Stages
1. `gather_sources` - Collect relevant documents
2. `analyze` - Extract key information
3. `synthesize` - Combine findings
4. `cite` - Track citations

### Constraints
- `citation_required: true` - All claims must cite sources

### Example
```bash
shad run "Summarize authentication best practices from my vault" \
  --vault ~/SecurityDocs \
  --strategy research
```

---

## Analysis Strategy

For data analysis and pattern detection.

### Required Stages
1. `data_collection` - Gather data points
2. `pattern_detection` - Identify patterns
3. `structured_output` - Format results

### Example
```bash
shad run "Analyze API usage patterns in my codebase examples" \
  --vault ~/CodeExamples \
  --strategy analysis
```

---

## Planning Strategy

For project planning and roadmap generation.

### Required Stages
1. `scope_definition` - Define boundaries
2. `dependency_mapping` - Identify dependencies
3. `milestone_generation` - Create milestones
4. `risk_assessment` - Identify risks

### Example
```bash
shad run "Create a migration plan for moving to microservices" \
  --vault ~/ArchDocs \
  --strategy planning
```

---

## Strategy Selection

Shad automatically selects strategies based on task keywords:

| Keywords | Strategy |
|----------|----------|
| build, create, implement, code | software |
| summarize, compare, analyze | research |
| patterns, data, metrics | analysis |
| plan, roadmap, migrate | planning |

Override with `--strategy`:
```bash
shad run "Create a summary" --strategy research  # Force research
```

---

## Custom Strategy Configuration

Strategies can be customized via vault metadata or CLI flags:

```bash
# Adjust recursion depth
shad run "..." --max-depth 4

# Adjust node limit
shad run "..." --max-nodes 100

# Adjust time limit
shad run "..." --max-time 600
```
