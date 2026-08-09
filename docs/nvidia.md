# Compress-LLM-Documentation Skill: Research Synthesis

Based on deep investigation across 20+ authoritative sources (Anthropic, Microsoft Research, ETH Zurich, Chroma, GitHub, prompt compression surveys 2024-2025), here's the definitive technical foundation for building a `compress-llm-documentation` skill.

---

## 1. Foundational Principles: How LLMs Read Differently Than Humans

| Human Reading | LLM Reading |
|---|---|
| Visual scanning, skimming, narrative flow | Token-by-token attention, no visual hierarchy |
| Infers context from "see above" / "as mentioned" | **Cannot resolve cross-references** without explicit restatement |
| Tolerates hedging ("you should consider...") | Needs **imperative, direct instructions** |
| Skims decorative formatting | Pays token-cost for every markdown token (bold, headers, HRs) |
| Navigates via TOC, sidebar | Navigates via **heading structure only** |
| Understands images/diagrams | **Ignores non-text content** (images, JS-rendered, screenshots) |

**Key insight**: LLMs treat documentation as *structured data*, not prose. Every token must earn its keep.

---

## 2. Two-Tier Compression Architecture (Lossless → Lossy)

Adopted from `markdown-compressor` skill (oborchers/fractional-cto) — **production-tested**.

### Tier 1: Lossless (20-40% reduction, zero semantic risk)
Apply in this order to avoid conflicts:

| # | Transformation | Token Savings | Rule |
|---|---|---|---|
| 1 | **HTML comment removal** | 2-5% | Strip `<!-- -->` entirely — invisible to LLMs |
| 2 | **Whitespace normalization** | 5-15% | Collapse 3+ blank lines → 1; trim trailing spaces; standardize 2-space indent |
| 3 | **Empty section removal** | 1-3% | Delete headings with no content (unless referenced) |
| 4 | **TOC removal** | 2-8% | LLMs navigate by heading hierarchy, not link lists |
| 5 | **Horizontal rule cleanup** | 1-2% | Keep only semantic boundaries (frontmatter); delete decorative `---` |
| 6 | **Redundant header consolidation** | 2-5% | Collapse parent headers with single child & no content |
| 7 | **Redundant emphasis reduction** | 1-3% | `**IMPORTANT:** ***NEVER***` → `**NEVER**` |
| 8 | **List marker standardization** | 1-2% | All bullets → `-`; 2-space nested indent; flatten cosmetic nesting |
| 9 | **Link simplification** | <1% | Collapse single-use reference links to inline |
| 10 | **Code block language tag normalization** | <1% | Remove `text`/`plaintext` tags when obvious from context |

**Never touch in lossless**: heading text, prose wording, code contents, table data, YAML frontmatter, semantic content.

---

### Tier 2: Lossy (40-70% reduction, requires reviewer loop)

Core compression principles with **before/after patterns**:

| Technique | Pattern | Reduction | Rule |
|---|---|---|---|
| **Imperative conversion** | "It's recommended you should validate..." → "Validate all input before processing." | ~50% | Strip: "It's important to", "Make sure to", "Remember to", "Please ensure". Start with verb. |
| **Cross-section deduplication** | Same rule in Overview/Database/Security → keep once in most relevant section, add one-line ref or delete section | 30-60% | If deduplication empties a section: (1) delete heading, (2) add one-line cross-ref, (3) merge into adjacent |
| **Prose-to-table** | Parallel descriptions → structured table with shared columns | ~40% | Error codes, config params, plan limits, API endpoints |
| **Inline consolidation** | Nested bullets → dash-separated: "LRU cache — max 1000 entries, 300s TTL, evict at 512MB" | 40-60% | When sub-items are short & uniform |
| **Implied knowledge deletion** | JWT explanation → "JWT auth. Tokens 1h, refresh 30d. Header: `Authorization: Bearer <token>`" | ~65% | Delete explanations of: REST, JSON, OAuth, MVC, async/await, generics, common patterns |
| **Example triage** | 5 snake_case examples → 1 inline: `snake_case: user_name, database_connection_string` | 70-90% | **Keep**: edge cases, non-obvious behavior, longest form. **Delete**: obvious applications. **Preserve fully**: operational examples (curl, CLI, config) — users copy-paste these. |
| **Section merging** | Input Validation + Data Sanitization → "Input Validation & Sanitization" | 30-50% | Merge when >50% content overlap |
| **Conditional compression** | If/else chains → decision tables or compact notation | 40-60% | "Free: 100/hr, Pro: 1K/hr, Enterprise: 10K/hr. Exceed → 429+Retry-After. >50% → block 1h" |
| **Boilerplate stripping** | "Welcome to our guidelines carefully crafted..." → delete entirely | 10-20% | Remove motivational, organizational preamble, disclaimers |
| **Standard abbreviations** | configuration→config, environment→env, authentication→auth, production→prod, repository→repo | 5-15% | Only when unambiguous in context |

---

## 3. Judgment Heuristics: What to KEEP vs REMOVE vs COMPRESS

| **KEEP (never remove)** | **REMOVE (safe)** | **COMPRESS (don't delete)** |
|---|---|---|
| Specific values/thresholds (timeout=30s) | Motivational filler ("This is important because...") | Valid instruction wrapped in hedging |
| Behavioral rules/prohibitions (NEVER, MUST NOT) | Restated info (same rule in intro + body) | Long example → short inline |
| Tool names, file paths, API endpoints | Hedging language ("you might want to consider") | Prose with useful specifics + fluff |
| Decision logic (if X then Y else Z) | Verbose transitions ("Now let's move to...") | Redundant emphasis (bold+caps+!) |
| Output format specs | Excessive illustrative examples (keep 1 distinctive) | |
| Edge case handling | HTML comments, decorative markdown | |
| Cross-references to other files/systems | Duplicate headers with no content | |
| Operational examples (copy-pasteable curl/CLI) | TOC, redundant HRs | |

---

## 4. Document Structure for LLM Consumption

### 4.1 Heading Hierarchy as Navigation System
- Headings = **anchor points**, not chapter markers
- Agent scans headings to orient: `## WHERE TO LOOK`, `## ANTI-PATTERNS`, `## CONTENT RULES`
- **No skipped levels** (h1→h3 flags structural issue)
- **Self-contained sections**: each `##` section must make sense without siblings

### 4.2 Tables > Prose for Mappings
```markdown
# Human-friendly (bad for LLMs)
Use POST for creating resources, GET for reading, PUT for full updates, PATCH for partial updates, DELETE for removal.

# LLM-friendly (survives compression)
| Operation | Method | Idempotent |
|-----------|--------|------------|
| Create    | POST   | No         |
| Read      | GET    | Yes        |
| Full update | PUT  | Yes        |
| Partial update | PATCH | No     |
| Delete    | DELETE | Yes        |
```
Tables survive summarization; prose gets compressed away.

### 4.3 Explicit Importance Signaling
Use keywords that survive LLM summarization (Anthropic compaction research):
- `CRITICAL`, `REQUIRED`, `MANDATORY`, `FORBIDDEN`, `NEVER`, `ALWAYS`
- `KEY:`, `NOTE:`, `WARNING:` as structured callouts
- Avoid: "important", "note that", "remember" — too weak

### 4.4 Section Length Guidelines (Promptless.ai + Chroma Context Rot)
| Doc Type | Max Tokens | Rationale |
|---|---|---|
| Reference (API, error codes) | ~22k words (30k tokens) | Lookup tasks degrade slowly |
| How-to / Tutorial | <16k tokens | Multi-step reasoning fails at 16-33k |
| Complex workflow (branching logic) | <8k tokens | Planning degrades fastest |
| AGENTS.md / CLAUDE.md | **150-200 lines** (ETH Zurich) | Beyond this → split into modular files |

---

## 5. Advanced Compression Techniques (Research-Backed)

### 5.1 Query-Aware Compression (LongLLMLingua / LLMLingua-2)
```python
# Microsoft LLMLingua - production ready
from llmlingua import PromptCompressor

compressor = PromptCompressor(
    model_name="microsoft/llmlingua-2-xlm-roberta-large-meetingbank",
    device_map="cuda",
    use_llmlingua2=True
)

compressed = compressor.compress_prompt(
    context=doc_sections,
    question=task_query,              # Query-aware compression
    rate=0.55,                        # Target compression ratio
    condition_in_question="after",    # Question guides what to keep
    reorder_context="sort",           # Minimize position bias
    dynamic_context_compression_ratio=0.3,
    condition_compare=True,           # Iterative token-level refinement
    force_tokens=["NEVER", "MUST", "API_KEY", "timeout"],  # Preserve critical tokens
)
```
**Results**: 4x fewer tokens, **+21.4% performance** on NaturalQuestions (LongLLMLingua paper).

### 5.2 Selective + Block Compression (Tool Documentation)
From "Concise and Precise Context Compression for Tool-Using LMs" (ACL 2024):
- **Selective**: Preserve key tokens (tool names, param names) as raw text; compress only prose
- **Block**: Chunk docs by fixed ratio → compress each chunk to fixed length → concatenate
- **Achieves 16x compression** with negligible performance loss on API-Bank/APIBench

### 5.3 AST-Based Code Compression (src2md)
For code-heavy docs (agents.md referencing codebase):
```python
from src2md import Converter

converter = Converter(
    target_tokens=100_000,
    summarization_levels={
        'critical': 'full',      # Full source (entry points, core types)
        'important': 'ast',       # AST: signatures, docstrings, type hints
        'supporting': 'minimal',  # Docstrings only
        'peripheral': 'exclude'   # Tests, generated, vendor
    }
)
```
Importance scoring: centrality (imports), complexity, recency, naming (`main.py`, `index.ts`).

### 5.4 Multi-Tier Document Architecture (WithAgents decision tree)
```
Does rule apply to EVERY session?     → CLAUDE.md (per-turn, ~2-5KB)
Does rule apply to SUBTREE only?      → AGENTS.md in that directory (per-dir)
Does rule apply to SPECIFIC TASK?     → SKILL.md with trigger phrases (per-trigger)
Otherwise                              → Inline code comment
```
**Token budget allocation**: Per-turn > Per-dir > Per-trigger > Inline

---

## 6. AGENTS.md / Agent-Specific Optimization (ETH Zurich + GitHub 2500 Repos)

### What WORKS (human-curated only):
| Include | Exclude |
|---|---|
| Custom build commands not elsewhere | Codebase overviews (agents discover independently) |
| Non-standard tooling (pixi vs pip) | Anything in README/existing docs |
| Behavioral constraints (NEVER commit secrets) | Architecture summaries |
| Hard boundaries (folders to never touch) | Redundant explanations |

### Structure (GitHub top-tier):
```markdown
# Project: <name> — <one-line purpose>

## Commands
| Task | Command |
|------|---------|
| Test | `pnpm test` |
| Typecheck | `pnpm typecheck` |
| Lint | `pnpm lint` |

## Testing
- Framework: Vitest
- Run: `pnpm test --run`
- Coverage threshold: 80%

## Code Style
- TypeScript strict mode
- ESLint + Prettier (config in repo)
- snake_case for variables, PascalCase for types

## Git Workflow
- Branch: `feat/<issue>-<slug>`
- Commit: Conventional Commits
- PR: squash merge, delete branch

## Boundaries (NEVER)
- Never commit `.env`, secrets, tokens
- Never modify `vendor/`, `dist/`, `node_modules/`
- Never add deps without discussion

## Project Structure (only if complex)
src/
├── core/       # Domain logic
├── api/        # Route handlers
└── infra/      # DB, cache, external
```

**Key finding**: LLM-generated AGENTS.md **hurts performance** (-3%, +20% cost). Human-curated gives +4% but still +19% cost. **Minimal is optimal**.

---

## 7. Compression Pipeline Design (Skill Implementation)

```
┌─────────────────────────────────────────────────────────────┐
│                    COMPRESS-LLM-DOCUMENTATION                │
├─────────────────────────────────────────────────────────────┤
│  INPUT: Markdown file(s) + context (task type, target model)│
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  STAGE 1: STRUCTURAL ANALYSIS (Pre-compression audit)       │
│  • Parse heading hierarchy, flag skipped levels             │
│  • Token count per section (words × 1.3 heuristic)          │
│  • Identify: empty sections, duplicate content, >500 token  │
│    sections, cross-ref patterns                             │
│  • Output: Analysis table + compression strategy recommendation
└─────────────────────────────────────────────────────────────┘
                            │
            ┌───────────────┴───────────────┐
            ▼                               ▼
┌───────────────────────┐       ┌───────────────────────┐
│  LOSSLESS PIPELINE    │       │  LOSSY PIPELINE       │
│  (Always safe)        │       │  (Reviewer loop)      │
├───────────────────────┤       ├───────────────────────┤
│  1. HTML comment strip│       │  1. Section splitter  │
│  2. Whitespace norm   │       │  2. Compressor agent  │
│  3. Empty section del │       │     (per technique)   │
│  4. TOC removal       │       │  3. Reviewer agent    │
│  5. HR cleanup        │       │     (diff + validate) │
│  6. Header consolidate│       │  4. User approve/edit │
│  7. Emphasis reduce   │       │  5. Merge back        │
│  8. List standardize  │       │                       │
│  9. Link simplify     │       │  Batch size: 5 secs   │
│  10. Code tag norm    │       │  Parallel: compressor │
│                       │       │  then reviewer        │
└───────────────────────┘       └───────────────────────┘
            │                               │
            └───────────────┬───────────────┘
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  STAGE 2: QUERY-AWARE REFINEMENT (Optional, for RAG/agent)  │
│  • If target task known: run LongLLMLingua with question    │
│  • Preserve force_tokens: CRITICAL, NEVER, paths, IDs       │
│  • Reorder sections: most relevant first (minimize pos bias)│
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  STAGE 3: OUTPUT & METRICS                                  │
│  • Compressed markdown + diff report                        │
│  • Token reduction: original vs compressed vs target        │
│  • Sections modified, techniques applied                    │
│  • Warning flags: info loss risk, oversized sections remain │
└─────────────────────────────────────────────────────────────┘
```

---

## 8. Non-Trivial / Original Tips

### 8.1 Pseudo-Code as Documentation
```markdown
## Rate Limiting Logic
```
IF plan == FREE:     limit = 100/hr
ELIF plan == PRO:    limit = 1000/hr
ELSE:                limit = 10000/hr

IF requests > limit:           return 429 + Retry-After
IF requests > limit * 1.5:     block_key(1h)
```
```
Denser than prose, unambiguous, survives compression, executable mentally.

### 8.2 Formulae for Constraints
```markdown
## Cache Sizing
max_entries = min(1000, memory_mb / 0.5)
ttl_seconds = 300
eviction_trigger = memory_mb > 512
```
LLMs parse math better than "cache holds up to 1000 entries or until memory exceeds 512MB..."

### 8.3 Structured Anti-Patterns (Negative Constraints)
```markdown
## ANTI-PATTERNS
| Pattern | Problem | Instead |
|---------|---------|---------|
| `SELECT *` | Full table scan | Explicit columns |
| `async` in loop | Sequential await | `Promise.all()` |
| `console.log` in prod | PII leakage | Structured logger |
```
Contrastive examples survive compression better than prose warnings.

### 8.4 Embedded Metadata for Compression Awareness
```markdown
## Authentication <!-- compression:preserve -->
JWT auth. Tokens 1h, refresh 30d. Header: `Authorization: Bearer <token>`.
<!-- compression:force_tokens=JWT,Authorization,Bearer,1h,30d -->
```
Compression pipeline reads HTML comments as directives.

### 8.5 Link Integrity via Anchor Contracts
```markdown
## Error Codes
See [Error Handling](#error-handling) for resolution patterns.

## Error Handling <!-- anchor:error-handling -->
| Code | Resolution |
|------|------------|
```
Compression preserves anchor links; validates cross-refs don't break.

### 8.6 Compression-Aware Authoring Patterns
| Pattern | Compression Result |
|---|---|
| `**KEY:**` + `**CRITICAL:**` → single `**CRITICAL:**` | Lossless emphasis reduction |
| 3 examples of same pattern → 1 inline | Lossy example triage |
| "As described in Configuration section" → delete (self-contained) | Deduplication |
| `configuration` → `config` (tech context) | Abbreviation |

---

## 9. Evaluation Metrics for Skill Quality

| Metric | Target | Measurement |
|---|---|---|
| **Lossless reduction** | 20-40% | Token count before/after |
| **Lossy reduction** | 40-70% | Token count + reviewer approval |
| **Semantic fidelity** | 100% (lossless) / >95% (lossy) | Reviewer agent + human spot-check |
| **Cross-ref integrity** | 0 broken | Automated link/anchor validation |
| **Critical token preservation** | 100% | Force-token checklist |
| **Heading structure validity** | No skipped levels | Markdown linter |
| **Operational example preservation** | 100% | Detect curl/CLI/config blocks |

---

## 10. Key References & Implementation Sources

| Resource | Type | Key Value |
|---|---|---|
| [Anthropic: Effective Context Engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) | Blog/Research | Compaction, memory, tool clearing primitives |
| [Microsoft LLMLingua](https://github.com/microsoft/LLMLingua) | Code + Paper | Query-aware compression, 4x tokens, +21% perf |
| [markdown-compressor skill](https://github.com/oborchers/fractional-cto/tree/main/markdown-compressor) | Production skill | Lossless/lossy techniques, reviewer loop, CLI |
| [src2md](https://github.com/queelius/src2md) | Code + Blog | AST-based code compression, multi-tier summarization |
| [Chroma Context Rot](https://www.trychroma.com/research/context-rot) | Research | Performance degrades >30k tokens, structural patterns matter |
| [Promptless: Agent Docs Guide](https://promptless.ai/blog/technical/agent-docs/) | Practical guide | Self-contained sections, error docs, example diversity |
| [ETH Zurich: Evaluating AGENTS.md](https://arxiv.org/html/2602.11988v1) | Paper | LLM-generated hurts; human minimal wins; 150-200 lines |
| [GitHub: 2500+ AGENTS.md lessons](https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/) | Blog | Six core areas, boundaries, commands early |
| [Fern: LLM-Friendly Docs](https://buildwithfern.com/post/how-to-write-llm-friendly-documentation) | Blog | llms.txt, content negotiation, 90% token reduction |
| [Prompt Compression Survey 2025](https://aclanthology.org/2025.naacl-long.368/) | Paper | Hard vs soft, token pruning, abstractive, extractive taxonomy |

---

## 11. Recommended Skill Configuration Schema

```yaml
# compress-llm-documentation skill config
version: "1.0"
target_model: "claude-3.5-sonnet"  # affects tokenizer estimation
mode: "lossy"                      # "lossless" | "lossy" | "auto"
target_reduction: 0.55             # 55% token reduction target
max_section_tokens: 500            # split threshold
force_preserve:
  - "NEVER"
  - "ALWAYS"
  - "MUST NOT"
  - "CRITICAL"
  - "REQUIRED"
  - "FORBIDDEN"
  - regex: "(API_KEY|SECRET|TOKEN|PASSWORD)"
  - regex: "\.md$|\.py$|\.ts$|\.json$|\.yaml$"
preserve_operational_examples: true
abbreviate_standard_terms: true
convert_prose_to_tables: true
merge_duplicate_sections: true
query_aware: false                 # enable if task context provided
reorder_for_relevance: false       # enable for RAG/agent contexts
output_format: "markdown"          # "markdown" | "json" | "diff"
reviewer_enabled: true
batch_size: 5
```

---

This synthesis gives you a production-ready foundation. The **markdown-compressor skill** is the closest existing implementation — study its `compress.md` command and `SKILL.md` for the exact agent orchestration patterns. Combine with **LLMLingua** for query-aware refinement when target tasks are known.
