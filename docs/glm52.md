Now let me compile everything into a comprehensive, actionable reference.

---

# Compress-LLM-Documentation Skill: Research & Engineering Guide

## 0. Executive Summary

This guide synthesizes findings from recent research (2024–2026), production tools, and benchmarks into actionable rules for compressing LLM-facing Markdown documentation (`agents.md`, `CLAUDE.md`, skills, memory files). Core thesis: **compression is not deletion — it's density maximization**. The goal is maximum semantic information per token (`SDE > 0.80`), placed at attention-optimal positions, with structure that survives chunking and retrieval.

---

## 1. Why Compress: The Cost of Unnecessary Context

### 1.1 Research Evidence

| Finding | Source | Impact |
|---------|--------|--------|
| Unnecessary requirements in context files **actively harm** agent performance, increase inference cost >20% | Gloaguen et al. (2026), [arxiv](https://arxiv.org/html/2602.11988v1) | -3% success rate (LLM-generated), +4% (minimal human-written) |
| Auto-generated `/init` context files **reduce** task success vs. no context | ETH Zurich study | Agents follow noise faithfully, broadening exploration |
| Semantic Density Effect: `SDE > 0.80` prompts outperform diluted ones by **+8.4pp** with 0 extra tokens | [arxiv 2604.17659](https://arxiv.org/abs/2604.17659) | Free accuracy gain via rewriting |
| Telegraph English symbolic rewriting: ~50% token reduction, **99.1% accuracy** preserved | [arxiv 2605.04426](https://arxiv.org/html/2605.04426) | Full semantic rewrite > token deletion |
| Lost in the Middle: U-shaped attention — middle content used dramatically worse | Liu et al. (2023), [agentpatterns.ai](https://agentpatterns.ai/context-engineering/lost-in-the-middle/) | Position is a real variable |

### 1.2 Quantified Compression Opportunities

| Content Type | Rule-Based Savings | Deep (LLM Rewrite) | Tool |
|-------------|-------------------|-------------------|------|
| LLM-generated prose | 8–9% | up to 58% | mdmin |
| README & long intros | 7–12% | 12–24% | mdmin |
| API & technical docs | 7–9% | 8–18% | mdmin |
| Docs-heavy repos (full tree) | 14% | 30–53% | mdcompress |
| HTML → Markdown conversion | 63–71% | — | formatarc |
| Query-conditioned extraction | 76–95% | — | SMELT, mdmin extract |

---

## 2. LLM-Readable vs. Human-Readable: Key Differences

### 2.1 Fundamental Differences

| Dimension | Human-Optimized | LLM-Optimized |
|-----------|----------------|---------------|
| **Redundancy** | Welcome (reinforces learning) | Harmful (dilutes attention, wastes tokens) |
| **Transitions** | Needed for flow ("As mentioned above…") | Zero-value noise tokens |
| **Tone/politeness** | Expected | Pure waste |
| **Examples** | Multiple, varied | One canonical, dense |
| **Structure** | Narrative flow | Atomic, self-contained chunks |
| **Length** | More = thorough | Less = higher signal-to-noise |
| **HTML** | Rich rendering | ~70% token tax vs Markdown |
| **Position** | Sequential reading | U-shaped attention (edges matter most) |

### 2.2 Format Token Economics

Measured on identical content (synthetic technical doc):

| Format | Characters | cl100k tokens | o200k tokens | chars/token |
|--------|-----------|--------------|-------------|-------------|
| HTML (rendered DOM) | 2,911 | 832 | 835 | 3.50 |
| Markdown (GFM) | 1,071 | 243 | 247 | 4.41 |
| Plain text | 986 | 213 | 217 | 4.63 |

Source: [formatarc.com](https://formatarc.com/en/blog/markdown-vs-html-for-llms/)

**Rule**: Always Markdown. Never HTML. YAML for nested data (beats XML by +80% token overhead on some models). XML only for section boundary tags, never for body content.

---

## 3. Compression Architecture: Multi-Layer Pipeline

Based on analysis of SMELT (4-layer), mdcompress (3-tier), mdmin (6-rule), and ContextCompressionEngine, the optimal pipeline is:

```
INPUT.md
  │
  ▼
┌─────────────────────────────────────────┐
│ Layer 0: FORMAT NORMALIZATION           │
│  HTML→MD, setext→ATX, unicode→ASCII     │
│  Strip frontmatter, comments, badges     │
├─────────────────────────────────────────┤
│ Layer 1: LOSSLESS STRUCTURAL (Safe)     │
│  Strip: TOC, HRs, metadata lines,       │
│  CTAs, tracking params, decorative      │
│  images, HTML wrappers, SEO chaff       │
│  Compact: tables→KV, collapse blanks    │
├─────────────────────────────────────────┤
│ Layer 2: SEMANTIC DENSITY (Aggressive)  │
│  Hedging removal: "in order to"→"To"    │
│  150+ verbose→telegraphic patterns      │
│  Cross-file dedup, phrase dictionary    │
│  Strip admonition prefixes, cross-refs   │
│  Dedup multilang examples                │
├─────────────────────────────────────────┤
│ Layer 3: RESTRUCTURING (Architectural)  │
│  Split → modular files + index          │
│  Position-aware: critical→poles         │
│  Atomic sections (1 concept per header) │
│  Code blocks: truncate mid-doc harder    │
├─────────────────────────────────────────┤
│ Layer 4: LLM REWRITE (Optional, Lossy)  │
│  Per-section rewrite with faithfulness  │
│  guard (threshold ≥0.95)                │
│  Telegraph English symbolic encoding     │
│  Pseudocode conversion for procedures    │
└─────────────────────────────────────────┘
  │
  ▼
COMPRESSED.md (token-optimized mirror)
  + original.md (retained for fallback)
```

---

## 4. Detailed Technique Catalog

### 4.1 Lossless Structural Rules (Tier 1 — Always Safe)

These remove zero-information tokens. Apply unconditionally.

```yaml
# Deterministic, lossless-to-meaning rules
strip_frontmatter: true        # YAML/TOML blocks → gone
strip_html_comments: true      # <!-- --> → gone
strip_badges: true             # shield.io images → gone
strip_decorative_images: true  # standalone <img> → gone
strip_horizontal_rules: true   # ---/===/___ → gone
strip_toc: true                # auto-generated TOC → gone
strip_metadata_lines: true     # "Last updated:", "Version:" → gone
strip_trailing_cta: true       # star/follow/sponsor → gone
strip_url_tracking_params: true # utm_* → gone
normalize_unicode: true        # smart quotes→ASCII, NBSP→space
strip_setext_headers: true     # ===/---  →  #/##
collapse_blank_lines: true     # 3+→2
compact_tables: true           # remove delimiter rows, trim whitespace
```

Source: [mdcompress rules](https://github.com/dhruv1794/mdcompress)

### 4.2 Table Compression (40–60% Token Reduction)

**Before** (Markdown pipe table — verbose):
```markdown
| Parameter | Type   | Required | Default | Description                          |
|-----------|--------|----------|---------|--------------------------------------|
| timeout   | int    | yes      | 30      | Request timeout in seconds           |
| retries   | int    | no       | 3       | Number of retry attempts             |
```

**After** (compact KV format):
```
timeout: int(req,def=30) # request timeout seconds
retries: int(opt,def=3)  # retry attempts
```

**After** (ultra-compact, for reference docs):
```
timeout: int!30    # req timeout sec
retries: int?3     # retry count
```
Notation: `!`=required, `?`=optional, value after `!`/`?`=default.

### 4.3 Verbose Pattern Removal (150+ Patterns)

Systematic telegraphic rewriting. A representative subset:

| Verbose | Telegraphic | Token Saved |
|---------|------------|-------------|
| In order to | To | 2 |
| Due to the fact that | Because | 4 |
| It is worth noting that | *(delete)* | 5 |
| As mentioned earlier | *(delete)* | 3 |
| It is important to note that | *(delete)* | 5 |
| In the context of | In | 3 |
| For the purpose of | For | 3 |
| With respect to | On | 3 |
| In the event that | If | 4 |
| A large number of | Many | 3 |
| Has the ability to | Can | 3 |
| Make a decision | Decide | 1 |
| Take into consideration | Consider | 3 |
| At this point in time | Now | 4 |
| In the near future | Soon | 3 |

Source: [mdmin](https://mdmin.dev/), [mdcompress](https://github.com/dhruv1794/mdcompress)

### 4.4 Dictionary Deduplication

Replace repeated multi-word phrases with short tokens:

```markdown
# Before (phrase appears 12×):
The authentication middleware validates JWT tokens...
The authentication middleware checks expiry...
The authentication middleware logs failures...

# After:
§1=authentication middleware
§1 validates JWT tokens
§1 checks expiry
§1 logs failures
```

Prepends a compact dictionary. Saves bytes; **caution**: can hurt token count under some tokenizers (SMELT finding). Test against actual tokenizer.

### 4.5 Cross-File Deduplication

For multi-file documentation trees (common in repos):

| Technique | What It Does |
|-----------|-------------|
| `strip-cross-file-dupes` | Replace exact duplicate sections with back-reference `[→ see ARCHITECTURE.md §3]` |
| `factor-cross-file-paragraphs` | Replace repeated prose with `§shared-setup` backref |
| `dedup-cross-file-code-blocks` | Replace repeated code blocks with `// → see src/example.ts` |
| `dedup-multilang-examples` | Collapse JS/Python/Go versions into one canonical + note |

### 4.6 Position-Aware Truncation

Non-trivial insight from mdcompress: **truncate middle-of-document code blocks more aggressively than head/tail blocks**.

```
Position in doc     | Truncation aggressiveness
─────────────────────┼──────────────────────────
First 30% (primacy)  | Minimal — high attention zone
30-70% (middle)      | Aggressive — low attention zone  
Last 30% (recency)   | Minimal — high attention zone
```

This aligns with the U-shaped attention curve. Code in the middle is less likely to be attended to, so truncating it there costs less.

---

## 5. Semantic Density Optimization

### 5.1 The SDE Formula

From [arxiv 2604.17659](https://arxiv.org/abs/2604.17659):

```
SDE(P) = S(P)/W(P) × (1 − R(P)) × C(P)

Where:
  S(P) = semantically loaded tokens (non-redundant, task-relevant)
  W(P) = total token count
  R(P) = redundancy fraction [0,1] — repeated concepts
  C(P) = concreteness score [0,1] — specific nouns, active verbs, 
         numbers, units, named entities
```

**Operational targets:**

| SDE Range | Class | Behavior |
|-----------|-------|----------|
| < 0.40 | Diluted | Hallucination-prone, unfocused |
| 0.40–0.65 | Standard | Normal performance |
| 0.65–0.80 | Dense | Above-average accuracy |
| > 0.80 | Ultra-Dense | +8.4pp accuracy, focused, low hallucination |

### 5.2 SDE Rewriting Heuristics

```
# Pseudocode for density-aware rewriting
function rewrite_for_density(text):
    # 1. Eliminate redundancy (R↓)
    text = remove_repeated_concepts(text)
    text = remove_transitional_phrases(text)  # "As mentioned", "As stated"
    text = remove_restated_context(text)
    
    # 2. Maximize concreteness (C↑)
    text = replace_abstract_with_specific(text)
      # "modern framework" → "React 18"
      # "fast response" → "<100ms latency"
      # "briefly" → "in 3 bullets"
      # "explain X" → "List 5 causes of X with dates"
    
    # 3. Remove noise tokens (S/W↑)
    text = remove_polite_preambles(text)  # "Can you please"
    text = remove_modal_softeners(text)   # "maybe", "kind of"
    text = remove_meta_commentary(text)   # "I want to know about"
    text = prefer_active_verbs(text)      # "derive" > "provide a derivation of"
    
    # 4. Quantify wherever possible
    text = add_units_quantities_entities(text)
    
    return text
```

### 5.3 Telegraph English (Symbolic Compression)

A full semantic rewrite protocol that decomposes prose into atomic fact-lines with ~40 relational symbols:

```
# Before (68 tokens):
"The application of heat to the metal caused it to expand, 
which resulted in a 2.3% increase in volume according to 
Smith et al. (2023)."

# After (14 tokens):
HEAT→METAL_EXPANSION
EXPANSION=2.3%vol
SRC:Smith2023
```

**Core symbol vocabulary:**

| Symbol | Meaning | Example |
|--------|---------|---------|
| `=` | Definition/equality | `VELOCITY=DISTANCE/TIME` |
| `→` | Causation/flow | `HEAT→EXPANSION` |
| `⇒` | Logical implication | `RAIN⇒WETNESS` |
| `∴` | Therefore | `X>Y ∧ Y>Z ∴ X>Z` |
| `∵` | Because | `FAILURE ∵ OVERLOAD` |
| `↑` / `↓` | Increase/decrease | `TEMP↑` |
| `∧` / `∨` / `¬` | And/or/not | `A∧B, ¬EVIDENCE` |
| `≈` / `≠` | Approximate/not equal | `COST≈USD10M` |
| `VS` | Contrast | `MODEL-A VS MODEL-B` |

**Tags**: `PAST:`, `NOW:`, `FUTURE:`, `LIKELY:`, `CONF=0.87`, `AGENT:`, `CTX:`, `DEF:`, `Q:/A:`

**Key principle**: Each line = one atomic fact → compression and semantic chunking become the same operation. Enables selective retrieval without additional processing.

Source: [arxiv 2605.04426](https://arxiv.org/html/2605.04426)

---

## 6. Structural Optimization: Splitting & Modularization

### 6.1 The Index Pattern (agents.md as Directory)

**Core principle** from [Red Hat Developer](https://developers.redhat.com/articles/2026/07/27/standardize-project-context-agentsmd-and-agent-skills): AGENTS.md should be an **index**, not a dump.

```markdown
# Project Name

> One-sentence description of what this project does.

## Toolchain
Lint: `pnpm lint` (Biome — see `biome.json`)
Test: `pnpm test` (Vitest)
Build: `pnpm build` (Vite)

## Context Map

| Topic | Document |
|-------|----------|
| Setup & CLI usage | `README.md` |
| System design | `docs/ARCHITECTURE.md` |
| Interface contracts | `specs/README.md` |
| Coding conventions | `docs/CONVENTIONS.md` |
| Agent personas | `.claude/skills/` |

## Judgment Boundaries
ALWAYS: Explain plan before writing code. Handle errors explicitly.
ASK: Before DB migrations, before deleting files.
NEVER: Commit secrets. Add deps without discussion. Guess on ambiguous specs.
```

**Size targets**: <150 lines (Red Hat), <200 lines (Claude). Smaller repos: 30–50 lines.

### 6.2 Atomic Section Principle

Every `##` or `###` heading must encapsulate **one atomic concept** that stands alone as a useful chunk:

```
✅ Good: "## Authentication via OAuth" → brief explanation + code example
❌ Poor: "## Getting Started" → 15 different concepts, no sub-headers
```

Each section should be **self-contained** — if retrieved in isolation by RAG, it still makes sense. Prepend heading path for disambiguation:

```markdown
# Setup > Authentication > OAuth > Configuration
[content...]
```

Source: [docmd.io](https://docs.docmd.io/guides/ai-optimisation/deterministic-chunkable-docs/), [samuelochoa.com](https://samuelochoa.com/expertise/rag/chunking/structure-aware)

### 6.3 md2idx Pattern: Index-Then-Retrieve

Instead of loading a 5,000-line spec, generate a 20-line index + sections array:

```
# Agent reads index first (20 lines):
npx md2idx spec.md | jq -r '.index'

# Then fetches only needed section (80 lines):
npx md2idx spec.md | jq -r '.sections[5]'

# Total: ~100 lines in context instead of 5,000
```

Source: [md2idx](https://github.com/oubakiou/md2idx)

### 6.4 Splitting Strategy for Large Files

| Criterion | Rule |
|-----------|------|
| **Boundary** | Heading changes (H1/H2/H3 shifts) |
| **Never split** | Code blocks, tables, lists — atomic units |
| **Too large** | Split on subheadings → sentences (never mid-sentence) |
| **Too small** | Merge with next section (same parent only) |
| **Token budget** | Count with actual model tokenizer (tiktoken) |
| **Heading context** | Prepend full heading path to each chunk |

---

## 7. Attention-Optimal Document Layout

### 7.1 The Sandwich Pattern

```
┌─────────────────────────────────┐
│ HIGH ATTENTION (Primacy)        │  ← Critical rules, NEVER constraints
│ First 30% of context            │  ← Executable commands
│                                 │  ← Stack definition
├─────────────────────────────────┤
│ LOW ATTENTION (Lost in Middle)  │  ← Reference material
│ 30-70% of context               │  ← Schemas, examples, lookup tables
│                                 │  ← Background context
│                                 │  ← Content agent will actively retrieve
├─────────────────────────────────┤
│ HIGH ATTENTION (Recency)        │  ← Task-specific instructions
│ Last 30% of context             │  ← Output format spec
│                                 │  ← Restate critical constraints
│                                 │  ← Acceptance criteria ("done" = ?)
└─────────────────────────────────┘
```

**Key insight**: Adding instructions in the middle pushes existing instructions further into the low-attention zone. Every unnecessary line **amplifies** the middle zone.

Source: [agentpatterns.ai](https://agentpatterns.ai/context-engineering/lost-in-the-middle/), [buecking/incontext](https://github.com/buecking/incontext/blob/main/docs/foundations/long-context-degradation.md)

### 7.2 The Pink Elephant Problem

> Telling an LLM what **not** to do puts that concept front-and-center in its attention mechanism.

```
❌ "Do not use tRPC"  →  token `tRPC` is now highly active
✅ "Use gRPC for inter-service calls"  →  positive constraint
```

**Rule**: Express constraints as positive directives. If you must state a negative, rephrase to avoid naming the forbidden thing.

Source: [asdlc.io AGENTS.md spec](https://asdlc.io/practices/agents-md-spec/)

---

## 8. Pseudocode & Symbolic Techniques in Documentation

### 8.1 Pseudocode for Procedures

Research shows LLMs follow pseudocode instructions **8–21% better** than natural language for compositional, constrained, and nested instructions.

Source: [arxiv 2505.18011](https://arxiv.org/html/2505.18011v2)

**Before** (prose, 45 tokens):
```markdown
When a user submits a form, first validate all required fields. If any 
field is missing, return a 400 error with the field names. If validation 
passes, check if the user already exists. If they do, update their record. 
If not, create a new user. Then send a welcome email.
```

**After** (pseudocode, 28 tokens):
```markdown
on_submit(form):
  validate(form.required) → missing? return 400{missing}
  user = find_user(form.email)
  user? update(user, form) : create_user(form)
  send_email(user, "welcome")
```

### 8.2 Decision Tables Instead of Prose

**Before** (prose, ~60 tokens):
```markdown
If the status is pending and the user is an admin, show the approve button.
If the status is pending and the user is not an admin, show a read-only view.
If the status is approved, show the published view. If the status is rejected,
show the rejection reason.
```

**After** (decision table, ~25 tokens):
```
status    user.role   → action
pending   admin       → show approve_btn
pending   *           → show readonly
approved  *           → show published
rejected  *           → show rejection_reason
```

### 8.3 Formula Compression

Express quantitative relationships as formulas rather than prose:

```markdown
# Before (30 tokens):
"The cache TTL is calculated by taking the base value of 300 seconds 
and multiplying it by the number of retry attempts, then dividing by 2."

# After (8 tokens):
TTL = 300 × retries / 2
```

### 8.4 Backreference Notation

For cross-referencing within compressed docs:

```
→ §3.2          # see section 3.2
→ @src/auth.ts  # see file
→ ⚑ critical    # flagged as critical
→ ⊕ optional    # optional section
```

---

## 9. Protected Facts Pattern

The most important principle for lossy compression: **identify facts that must never be lost and protect them from summarization**.

```
# Protected facts block — survives every compression pass
╔══ PROTECTED ════════════════════════════════╗
║ user_id = u_123                             ║
║ api_endpoint = https://api.example.com/v2   ║
║ rate_limit = 1000 req/min                   ║
║ auth = Bearer token, refresh @ 3600s        ║
║ NEVER: commit .env files                    ║
╚════════════════════════════════════════════╝
```

**What to protect**: identifiers, exact numbers, names, explicit decisions, state that later steps depend on. These are carried forward **verbatim**, never paraphrased.

Source: [adaptiverecall.com](https://www.adaptiverecall.com/context-engineering/compress-context.php)

---

## 10. The Toolchain-First Principle

> If a constraint can be enforced by a linter, formatter, type checker, hook, or CI gate, it **must not** be restated in agents.md.

| Type | Example | Home |
|------|---------|------|
| Toolchain-enforced | no `var`, import order, formatting | `biome.json` / `eslint` / `tsconfig` |
| Judgment / architectural | prefer composition, ask before adding deps | `agents.md` |
| Session-scoped persona | Critic, Builder | skill/workflow file |
| Task-specific style | API naming for this module | spec/PBI |

Source: [asdlc.io](https://asdlc.io/practices/agents-md-spec/)

**Litmus test for every line**: "Would removing this cause the agent to make a mistake it wouldn't otherwise make?" If not → delete.

---

## 11. Compression Skill: Recommended Rule Set

### 11.1 Detection & Classification Phase

```python
# Pseudocode for compression skill
def classify_content(text):
    sections = parse_markdown_ast(text)
    for section in sections:
        section.type = classify(section)
        # Types: CODE | CONFIG | TABLE | PROSE | REFERENCE | 
        #        COMMAND | BOUNDARY | METADATA | BOILERPLATE
        
        section.density = compute_sde(section.text)
        section.position = compute_position(section, doc_length)
        # position ∈ {PRIMACY, MIDDLE, RECENCY}
        
        section.compress_strategy = decide(section)
        # STRATEGIES:
        #   KEEP_VERBATIM    — code, configs, tables, commands, boundaries
        #   TELEGRAPH        — prose, explanations, descriptions
        #   SYMBOLIC_REWRITE — procedures, decision logic, relationships
        #   STRIP            — boilerplate, metadata, chaff, duplicates
        #   TRUNCATE         — large code blocks in MIDDLE zone
        #   SPLIT            — oversized sections → modular files
```

### 11.2 Compression Decision Matrix

| Content Type | Position | Strategy | Expected Savings |
|-------------|----------|----------|-----------------|
| Code blocks | PRIMACY/RECENCY | KEEP_VERBATIM | 0% |
| Code blocks | MIDDLE | TRUNCATE (configurable) | 20–60% |
| Config/YAML | Any | KEEP_VERBATIM | 0% |
| Tables | Any | COMPACT → KV format | 40–60% |
| Prose explanations | Any | TELEGRAPH (150+ patterns) | 8–35% |
| Procedures | Any | SYMBOLIC_REWRITE (pseudocode) | 40–60% |
| Boilerplate (CTA, badges, TOC) | Any | STRIP | 100% |
| Duplicated across files | Any | BACKREF | 100% |
| Hedging/transitions | Any | STRIP | 100% |
| Architecture description | Any | SPLIT → index + linked file | 80–95% |

### 11.3 Faithfulness Guard

After compression, run a faithfulness audit:

```yaml
eval:
  backend: ollama          # or anthropic, openai
  model: llama3.1:8b       # cheap judge model
  threshold: 0.95          # reject if below
  questions_per_doc: 10    # factual Q&A against original
  # Generates questions from original, tests compressed version
  # answers them. If accuracy < threshold → rollback section.
```

Source: [mdcompress eval](https://github.com/dhruv1794/mdcompress)

---

## 12. Non-Trivial & Original Tips

### 12.1 Hybrid Storage (Compressed + Original)

Keep compressed version in active context, full version in external storage. If a later question needs a dropped detail → re-fetch original.

```
context = {
    active: compressed_doc,      # in LLM context window
    storage: original_doc,        # on disk, retrievable
    fallback: "if compressed insufficient, fetch @original"
}
```

This makes compression **reversible** — the safest form.

### 12.2 Prompt Caching Awareness

For Claude Enterprise: `CLAUDE.md` is cached (content-addressed). First request pays full price; subsequent within ~5min pay cache-read rate. **Any change invalidates cache**.

**Implication**: Keep volatile content (session state, task specifics) **separate** from stable content (project rules, conventions). Only stable content benefits from caching.

Source: [Claude Help Center](https://support.claude.com/en/articles/14553240-give-claude-context-claude-md-and-better-prompts)

### 12.3 Negative Constraints via Positive Redirection

```
❌ "Never use var"              → token "var" activated
✅ "Use let/const exclusively"  → positive, no forbidden token

❌ "Don't use axios"            → "axios" activated  
✅ "Use fetch() for HTTP"       → positive redirection
```

### 12.4 One Rule Per Line, One Concept Per Line

```
# Bad (compound, 3 concepts in 1 sentence):
"Always use async/await for async operations, handle errors with 
try/catch, and never use .then() chains."

# Good (atomic, 3 lines):
use async/await for async ops
handle errors via try/catch
prefer async/await over .then()
```

### 12.5 Attach "Why" to Rules

Claude generalizes better when it knows the reason:

```
# Bad:
"Split functions over 50 lines"

# Good:
"Split functions >50 lines  # keeps diff reviewable + testable"
```

### 12.6 Measurable Form

```
❌ "Write clean code"
✅ "Split functions >50 lines"
✅ "Name functions verb+noun: getUserData, calculateTotal"
✅ "Max 3 params per function"
```

### 12.7 Use Emoji as Semantic Tokens (Sparingly)

Emoji can function as compact semantic tags that LLMs parse well:

```
🚫 NEVER: commit secrets, edit node_modules/
⚠️ ASK: DB migrations, adding deps
✅ ALWAYS: run tests before commit, follow naming conventions
📌 PIN: api_endpoint, rate_limit (protected facts)
⊕ OPTIONAL: see docs/ADVANCED.md
```

1-2 tokens per emoji vs 5-10 tokens for "**Warning:**" prefix.

### 12.8 Code Block Annotation Compression

```markdown
# Before:
```typescript
// This function fetches user data from the API
// It takes a user ID string and returns a Promise<User>
// It throws an error if the ID is empty
async function fetchUserById(id: string): Promise<User> {
```

# After (strip comments, let types speak):
```typescript
async function fetchUserById(id: string): Promise<User> {
```

The type signature already communicates everything the comments said.
```

### 12.9 Escaping the "Truncation Cliff"

> The biggest factor in whether an agent sees your content isn't where it falls on the page. It's whether the page fits within the agent's truncation limits at all.

A 400K-character HTML page → agent sees ~25%. A focused 5K-character Markdown page → read in full.

**Rule**: The fix isn't reordering within a long page. The fix is making the page shorter, or serving a focused version.

Source: [dacharycarey.com](https://dacharycarey.com/2026/03/12/vibes-out-data-in/)

---

## 13. Recommended Tool Stack for the Skill

| Tool | Role | URL |
|------|------|-----|
| **mdmin** | Rule-based compression, extract, context budget | [mdmin.dev](https://mdmin.dev/) |
| **mdcompress** | 35 deterministic rules, 3 tiers, faithfulness audit, MCP | [github.com/dhruv1794/mdcompress](https://github.com/dhruv1794/mdcompress) |
| **SMELT** | 4-layer: archival→semantic→macro→query-conditioned | [github.com/TooCas/SMELT](https://github.com/TooCas/SMELT/) |
| **md2idx** | Split → index + sections for on-demand retrieval | [github.com/oubakiou/md2idx](https://github.com/oubakiou/md2idx) |
| **llm-docs-builder** | HTML→MD, token counts, priority labels | [github.com/mensfeld/llm-docs-builder](https://github.com/mensfeld/llm-docs-builder) |
| **split-markdown4gpt** | Token-bounded chunking respecting AST | [github.com/twardoch/split-markdown4gpt](https://github.com/twardoch/split-markdown4gpt) |
| **tiktoken** | Token counting (must match model tokenizer) | [github.com/openai/tiktoken](https://github.com/openai/tiktoken) |

---

## 14. Key References

| # | Resource | URL |
|---|----------|-----|
| 1 | AGENTS.md Specification (research-backed) | [asdlc.io](https://asdlc.io/practices/agents-md-spec/) |
| 2 | Red Hat: Standardize project context with AGENTS.md | [developers.redhat.com](https://developers.redhat.com/articles/2026/07/27/standardize-project-context-agentsmd-and-agent-skills) |
| 3 | GitHub Blog: agents.md lessons from 2,500 repos | [github.blog](https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/) |
| 4 | Claude Help: CLAUDE.md best practices | [support.claude.com](https://support.claude.com/en/articles/14553240-give-claude-context-claude-md-and-better-prompts) |
| 5 | Evaluating AGENTS.md (Gloaguen et al. 2026) | [arxiv.org/abs/2602.11988](https://arxiv.org/html/2602.11988v1) |
| 6 | Semantic Density Effect (SDE) | [arxiv.org/abs/2604.17659](https://arxiv.org/abs/2604.17659) |
| 7 | Telegraph English symbolic compression | [arxiv.org/html/2605.04426](https://arxiv.org/html/2605.04426) |
| 8 | Lost in the Middle (attention U-shape) | [agentpatterns.ai](https://agentpatterns.ai/context-engineering/lost-in-the-middle/) |
| 9 | Long-context degradation deep dive | [github.com/buecking/incontext](https://github.com/buecking/incontext/blob/main/docs/foundations/long-context-degradation.md) |
| 10 | Markdown vs HTML for LLMs (token benchmarks) | [formatarc.com](https://formatarc.com/en/blog/markdown-vs-html-for-llms/) |
| 11 | Language, format, placement for LLMs | [tellian.io](https://tellian.io/2026/06/21/language-format-placement/) |
| 12 | Context compression without losing signal | [adaptiverecall.com](https://www.adaptiverecall.com/context-engineering/compress-context.php) |
| 13 | Structure-aware chunking | [samuelochoa.com](https://samuelochoa.com/expertise/rag/chunking/structure-aware) |
| 14 | Deterministic & chunkable docs | [docs.docmd.io](https://docs.docmd.io/guides/ai-optimisation/deterministic-chunkable-docs/) |
| 15 | llms.txt specification | [llmstxt.org](https://llmstxt.org/) |
| 16 | Training with Pseudo-Code (8–21% improvement) | [arxiv.org/abs/2505.18011](https://arxiv.org/html/2505.18011v2) |
| 17 | Pseudocode prompting (EMNLP 2023) | [aclanthology.org/2023.emnlp-main.939](https://aclanthology.org/2023.emnlp-main.939/) |

---

## 15. Summary: The 10 Commandments of LLM Doc Compression

1. **Density over brevity** — Maximize SDE > 0.80. A 60-token prompt can be denser than a 15-token one if every token carries specific, non-redundant, concrete meaning.
2. **Lossless first, lossy second** — Exhaust structural/whitespace/dedup rules before any LLM rewrite. Never compress protected facts.
3. **Position is a variable** — Critical rules at poles (first/last 30%), reference material in middle. Adding lines amplifies the weak middle.
4. **Index, don't dump** — AGENTS.md is a directory, not a spec. Point to files. Keep <150 lines.
5. **Toolchain first** — If a linter enforces it, it doesn't belong in agents.md. The tool IS the constraint.
6. **Positive constraints** — Avoid the Pink Elephant Problem. Say what to do, not what not to do.
7. **Atomic sections** — One concept per heading. Self-contained chunks that survive retrieval.
8. **Pseudocode > prose** — For procedures, decisions, and logic: 40–60% token savings + better instruction following.
9. **Symbolic encoding** — Telegraph English for fact-dense content: atomic lines, relational symbols, ~50% reduction at 99% fidelity.
10. **Retain the original** — Hybrid storage: compressed in context, full on disk. Compression becomes reversible.