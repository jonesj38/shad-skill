# Shad Strategies

Strategies define how Shad decomposes complex tasks into manageable subtasks. Each strategy provides a **skeleton** with required stages, optional stages, and constraints.

## How Strategies Work

1. **Strategy Selection**: Heuristic classifier or user override
2. **Skeleton Loading**: Required/optional stages loaded
3. **LLM Refinement**: LLM fills in task-specific details
4. **Constraint Enforcement**: Invariants checked during execution

The LLM can:
- Add/remove optional nodes
- Split implementation into specific modules
- Add soft dependencies
- Cannot violate required stages without explicit waiver

---

## Software Strategy

For building applications and generating code.

### Skeleton

```yaml
required:
  - clarify_requirements
  - project_layout
  - types_contracts
  - implementation
  - verification
  - synthesis
optional:
  - db_schema
  - auth
  - openapi
  - migrations
  - docs
constraints:
  - contracts_first: true
  - imports_must_resolve: true
  - no_implicit_writes: true
```

### Required Stages
1. `clarify_requirements` - Understand what needs to be built
2. `project_layout` - Define file structure
3. `types_contracts` - Define interfaces first (contracts-first)
4. `implementation` - Generate code
5. `verification` - Check syntax, types, tests
6. `synthesis` - Assemble final output

### Key Features
- **Contracts-first**: Types node runs before implementation
- **Two-pass import resolution**: Build export index, then generate implementations
- **File manifests**: Structured output with paths, content, metadata

### Example
```bash
shad run "Build a user management API" \
  --vault ~/APIDocs \
  --strategy software \
  --verify strict \
  --write-files --output ./api
```

---

## Research Strategy

For analysis, summarization, and synthesis tasks.

### Skeleton

```yaml
required:
  - clarify_scope
  - gather_sources
  - synthesize
  - cite
optional:
  - compare_perspectives
  - identify_gaps
constraints:
  - must_cite_vault: true
  - max_claims_per_source: 5
```

### Required Stages
1. `clarify_scope` - Define research boundaries
2. `gather_sources` - Collect relevant documents
3. `synthesize` - Combine findings
4. `cite` - Track citations

### Key Features
- **Citation tracking**: All claims must cite vault sources
- **Multi-source synthesis**: Combines information across documents
- **Perspective comparison**: Optional stage for balanced analysis

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

### Key Features
- **Pattern detection**: Identifies recurring themes
- **Structured output**: Consistent formatting for results

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

### Key Features
- **Dependency mapping**: Understands task relationships
- **Risk assessment**: Identifies potential blockers

### Example
```bash
shad run "Create a migration plan for moving to microservices" \
  --vault ~/ArchDocs \
  --strategy planning
```

---

## Strategy Selection

### Automatic Selection

Shad uses a heuristic classifier (no LLM call) based on task keywords:

| Keywords | Strategy | Confidence |
|----------|----------|------------|
| build, create, implement, code, api | software | High |
| summarize, compare, review, research | research | High |
| analyze, patterns, data, metrics | analysis | Medium |
| plan, roadmap, migrate, schedule | planning | Medium |

**Decision rule:**
- confidence >= 0.7 -> proceed with guess
- confidence < 0.7 -> default to `analysis`, allow LLM confirmation

### Manual Override

```bash
shad run "Create a summary" --strategy research  # Force research
```

### Mid-Run Switch

The LLM can emit `strategy_switch_request` with evidence if it determines a different strategy would be more appropriate.

---

## Budget & Limits Configuration

Strategies can be customized via CLI flags:

```bash
# Adjust recursion depth
shad run "..." --max-depth 4

# Adjust node limit
shad run "..." --max-nodes 100

# Adjust time limit (seconds)
shad run "..." --max-time 600

# Adjust token budget
shad run "..." --max-tokens 200000
```

### Default Budgets

| Budget | Default | Enforcement |
|--------|---------|-------------|
| `max_depth` | 3 | Checked before decomposition |
| `max_nodes` | 50 | Checked before creating nodes |
| `max_wall_time` | 300s | Checked periodically |
| `max_tokens` | 100,000 | Atomic Redis deduction |

---

## Soft Dependencies & Context Packets

Decomposition emits both:
- **Hard deps**: Must complete first
- **Soft deps**: Useful if available (don't block execution)

When a node completes, it produces a **context packet** (summary, artifacts, keywords) that can be injected into pending nodes' retrieval.

This enables:
- Parallel execution where possible
- Cross-subtask context sharing
- Efficient use of compute budget
