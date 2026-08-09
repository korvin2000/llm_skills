Based on my research, here are the most efficient, practical, and modern recommendations for compressing and optimizing LLM documentation (agents.md files, markdown memory files, etc.):

## Core Principles for LLM-Optimized Documentation

### 1. **Structure & Format Fundamentals**

| Principle | Implementation | Why It Matters |
|-----------|---------------|----------------|
| **Answer-first content** | Lead with direct answer/context-setting paragraph | LLMs extract first 500 tokens; must contain product, version, surface |
| **Self-contained sections** | Treat each chunk as if it's the only page seen | Avoids "see above" references that cause hallucinations |
| **Semantic headings** | Write headings as questions developers would ask | Better retrieval than category labels like "Authentication" |
| **Runnable code blocks** | Include imports, realistic values, copy-paste ready | Agents copy examples literally; fragments break integrations |
| **Atomic pages** | One capability per page with complete context | Prevents fragmented retrieval across multiple chunks |

### 2. **AGENTS.md Compression Techniques**

**Target**: ~2.5k tokens (most files are 10k+ unnecessarily)

**Compression tiers:**
1. **Safe (Tier-1)**: Deterministic, lossless-to-meaning rules
2. **Aggressive (Tier-2)**: Prose simplification (14-53% reduction)
3. **LLM (Tier-3)**: Section-level rewriting with faithfulness guard

**Key compression rules:**
```markdown
# Before (640 tokens)
## Authentication
Our application uses JWT-based authentication. When a user logs in, the server generates a JWT token...

# After (120 tokens) 
[Auth]|JWT in httpOnly cookie|24h expiry
|middleware: extract token > verify > attach user to req
|login: validate creds > generate JWT(id,role) > set cookie
```

**Techniques that work:**
- **Pipe-delimited indexes**: `routing:{defining,dynamic,middleware}` (~75% reduction)
- **Single-line directives**: `imports: builtin > external > internal > types` (~90% reduction)
- **Abbreviated keys**: `env: req str="prod"` instead of `Required: true, Type: string, Default: "prod"`
- **Brace expansion**: `{Button,Input,Modal}.tsx` instead of separate entries

### 3. **What to Include vs Exclude**

| Include | Exclude | Reason |
|---------|---------|--------|
| Non-obvious commands with flags | Commands in package.json | Agents already read package.json |
| Rules differing from defaults | Standard language conventions | Agents know PEP 8/Prettier defaults |
| Architecture constraints | Full API documentation | Link to docs, don't embed |
| Explicit boundaries | "Write clean code" | Wastes context; agent already tries |
| Project-specific gotchas | Info duplicated from README | Redundancy reduces performance 23% |

### 4. **Context Compression Strategies**

**Three-layer approach:**
1. **Selection**: Keep chunks verbatim (retrieval, keyword filters)
2. **Transformation**: Replace with summaries, outlines, tables
3. **Token pruning**: Remove low-information tokens (LLMLingua, whitespace normalization)

**Practical techniques:**
- **Hierarchical map-reduce**: Chunk → summarize each → merge globally
- **Structured extraction**: Convert prose to JSON, tables, AST snippets
- **Token budget allocation**: 
  - System + tools: 4-12K
  - Retrieved docs: 40-80K  
  - Conversation: 8-24K
  - User message: 1-8K
  - Reserved output: 4-16K

### 5. **LLM vs Human-Optimized Formats**

| Aspect | LLM-Optimized | Human-Optimized |
|--------|---------------|-----------------|
| **Structure** | Flat, self-contained chunks | Hierarchical with navigation |
| **References** | No forward/backward references | "See above" acceptable |
| **Examples** | Copy-paste runnable snippets | May need context |
| **Headings** | Questions/answers | Category labels |
| **Metadata** | Frontmatter (title, description, date) | Often omitted |
| **Code blocks** | Complete with imports | Fragments acceptable |
| **Diagrams** | Text-based (Mermaid) or paired prose | Images alone acceptable |

### 6. **Non-Trivial Optimization Techniques**

**Advanced compression methods:**
- **Cross-file deduplication**: Replace duplicate sections with back-references
- **Phrase dictionary factorization**: Create glossary preamble for repeated multi-word phrases
- **Position-aware budget**: Truncate middle sections more aggressively than head/tail
- **Pseudocode formulas**: Use mathematical notation for complex logic
- **Multilingual snippets**: Leverage code comments in different languages for context
- **ASCII diagrams**: Replace images with text-based representations
- **Structured memory slots**: Replace raw transcripts with `intent|changes|decisions|next_steps`

**Example structured memory:**
```markdown
[Task]|Refactor monorepo auth system
[Changes]|1. Extracted JWT logic to /lib/auth.ts
          2. Added middleware validation
[Decisions]|1. Keep httpOnly cookies (security requirement)
           2. No refresh tokens (product decision)
[Next]|1. Update API routes to use new middleware
      2. Add integration tests
```

### 7. **Modern Infrastructure for LLM Documentation**

**Essential artifacts:**
- `/llms.txt`: Curated markdown index
- `/llms-full.txt`: Concatenated all pages
- `/.md` endpoints: Per-page raw markdown
- `sitemap.md`: Semantic sitemap with descriptions
- MCP servers: Structured query access

**Frontmatter schema:**
```yaml
---
title: "Deploy to Production"
description: "Deploy app to staging or production environments"
category: "deployment"
date: "2026-08-09"
updated: "2026-08-09T10:30:00Z"
---
```

### 8. **Research-Backed Insights**

- **28.6% faster**, **16.6% fewer tokens** with well-written AGENTS.md (Princeton study)
- **32 KiB default size cap** (Codex) - content beyond silently truncated
- **Auto-generated files reduce success 2%** and increase cost 23% (human-written better)
- **Context rot**: Performance degrades meaningfully at 30k+ tokens
- **Passive context beats active retrieval**: 100% vs 53% eval pass rate

### 9. **Implementation Checklist**

1. **Start with 20-30 lines** covering what agents most often get wrong
2. **Apply compression tiers** based on document type
3. **Audit faithfulness** - ensure compressed docs answer factual questions identically
4. **Use consistent structure**: Overview → Commands → Architecture → Rules → Learnings
5. **Date entries** - add `## Learnings` with dates from actual issues
6. **Link, don't embed** - reference external docs instead of including full content
7. **Test with target LLMs** - verify compression doesn't break functionality

### 10. **Tools & Resources**

- `mdcompress`: 35-rule compression with faithfulness audit
- `prompt-compression`: 8 compression rules with examples
- `llmlingua`: Token-level compression (2-5x reduction)
- `SKILL.md` frontmatter: `name|description|disable-model-invocation|allowed-tools`

**Key formula**: 
```
Compression = Selection + Transformation + Pruning
Selection: Keep relevant chunks verbatim
Transformation: Summarize/structure remaining
Pruning: Remove low-value tokens
```

This framework provides a comprehensive basis for creating a `compress-llm-documentation` skill that can transform verbose documentation into LLM-optimized formats while preserving essential information and functionality.


Based on my deep research across the latest academic papers, industry tools, and specifications, here are the most advanced, practical, and non-trivial recommendations for compressing and optimizing LLM documentation:

## Research Foundations & Key Papers

### 1. **Theoretical Framework: Rate-Distortion Theory for Context Compaction**

The most important theoretical breakthrough is the unification of context compression as a **rate-distortion problem**:

| Paper | Key Insight | Link |
|-------|-------------|------|
| **"What to Keep, What to Forget"** (July 2026) | All memory compaction (KV cache, prompts, agent memory) is one optimization problem: maximize I(M;A|Q) subject to I(M;X) ≤ B | [arxiv.org/abs/2607.08032](https://arxiv.org/abs/2607.08032) |
| **Context Compaction Theory** (Aug 2026) | Context Generation (summarization) can need strictly less budget than Context Selection (chunking) for some queries | [arxiv.org/abs/2608.01326](https://arxiv.org/abs/2608.01326) |
| **"Remember the Decision, Not the Description"** | Optimal compression preserves decision-relevant bits, not descriptive bits | [arxiv.org/abs/2605.10870](https://arxiv.org/abs/2605.10870) |

**Key formula**: 
```
Error ≥ H(Y|Q) - (B-1)/log|Y| when B < I*(Q)
```
Where I*(Q) = task-conditioned information content. **Below this bound, every method must fail.**

### 2. **Three Compression Families Comparison**

From SurePrompts' 2026 analysis ([sureprompts.com/blog/context-compression-techniques](https://sureprompts.com/blog/context-compression-techniques)):

| Dimension | Summarization | Semantic Chunking | Token-Level |
|-----------|---------------|-------------------|-------------|
| **Operation** | Rewrite | Select | Trim |
| **Compression Rate** | High, tunable | Very high for narrow queries | Moderate, stackable |
| **Fidelity Loss** | Holistic drops | Threshold cliff | Local, unpredictable |
| **Best For** | Chat history, multi-session state | Large corpora with specific queries | Final pass on prose |
| **Worst For** | Verbatim material | Broad queries | Exact wording tasks |

**Hybrid Stack** (most production systems):
1. Summarize then chunk (rolling summary + verbatim tail)
2. Chunk then compress (retrieval + token-level trimming)
3. Structured summary + verbatim tail

## Advanced AGENTS.md Optimization

### 3. **Research-Backed AGENTS.md Specification**

From the ASDLC.io spec ([asdlc.io/practices/agents-md-spec](https://asdlc.io/practices/agents-md-spec)):

**Core Philosophy**:
- **Minimal by Design**: If constraint can be expressed elsewhere, it must not live here
- **Toolchain First**: Restating linter rules creates maintenance debt and dilutes signal
- **Pink Elephant Problem**: Telling LLM what *not* to do ensures concept is front-and-center
- **20-30 lines max** for smaller repos, 30-50 for medium, <150 for large

**Critical Research Finding** (Gloaguen et al., 2026 on 138 repos):
- **LLM-generated context files reduce performance** by 2-3% while increasing cost by 20-23%
- **Developer-written files improve** by only 4% marginally
- **Mechanism**: Agents follow instructions faithfully, broadening exploration without improving outcomes

**AGENTS.md Structure**:
```markdown
# AGENTS.md

> **Project:** High-throughput gRPC service for real-time financial transactions.  
> **Core constraints:** Zero-trust security model, ACID compliance.

## Toolchain
| Action | Command | Authority |
|---|---|---|
| Build | `make build` | Outputs to `./bin` |
| Test | `make test` | Runs with `-race` detector |
| Lint | `golangci-lint run` | See `.golangci.yml` |

## Judgment Boundaries
**NEVER**
- Commit secrets, tokens, or `.env` files
- Add external dependencies without discussion

**ASK**
- Before adding external dependencies
- Before running database migrations

**ALWAYS**
- Explain your plan before writing code
```

## Practical Compression Tools & Techniques

### 4. **Tool-Specific Compression Methods**

| Tool | Approach | Savings | Link |
|------|----------|---------|------|
| **mdcompress** | 35 deterministic rules, faithfulness audit, MCP server | 14-53% on docs-heavy repos | [github.com/dhruv1794/mdcompress](https://github.com/dhruv1794/mdcompress) |
| **mdmin** | 150+ verbose patterns, TF-IDF extraction, ContextBudget manager | 13-35% rule-based, 70-95% extraction | [mdmin.dev](https://mdmin.dev/) |
| **LLMLingua** | Token-level compression using small LM perplexity | 2-5x with modest quality loss | [microsoft.com/en-us/research/project/llmlingua](https://www.microsoft.com/en-us/research/project/llmlingua/) |
| **LLMLingua-2** | Data distillation + BERT-level encoder | 3-6x faster than LLMLingua | [ACL Anthology 2024](https://aclanthology.org/2024.findings-acl.57/) |

**mdcompress Tiers**:
- **Tier-1 (Safe)**: Strip frontmatter, normalize unicode, remove badges, compress code blocks
- **Tier-2 (Aggressive)**: Cross-file deduplication, phrase dictionary factorization, strip hedging phrases
- **Tier-3 (LLM)**: Section-level rewriting with faithfulness guard

### 5. **Non-Trivial Compression Techniques**

**From research and tools**:

1. **Position-Aware Budget**: Truncate middle sections more aggressively than head/tail (memory primacy/recency effect)
2. **Dictionary Deduplication**: Replace repeated multi-word phrases with §1, §2 tokens + compact dictionary preamble
3. **Table Compression**: Convert pipe tables to CSV/key:value (40-60% reduction)
4. **Cross-File Deduplication**: Replace duplicate sections across repo files with back-references
5. **Code Block Preservation**: Always protect code blocks/inline code during compression
6. **Semantic Preservation Loss**: Train compressor with semantic loss to retain key meanings (Nano-Capsulator)

**LLM-Optimized Format Rules**:
```markdown
# Answer-first structure
## What it does (direct answer)
### How to use it (runnable example)
### When to use it (constraints)
```

## LLM vs Human Documentation Design

### 6. **Three Layers of Agent Readiness**

From Vercel's comprehensive guide ([vercel.com/kb/guide/make-your-documentation-readable-by-ai-agents](https://vercel.com/kb/guide/make-your-documentation-readable-by-ai-agents)):

| Layer | Mechanism | Purpose |
|-------|-----------|---------|
| **Discovery** | `/llms.txt`, `sitemap.md`, JSON-LD structured data | Entry point for agents |
| **Retrieval** | Content negotiation, `.md` endpoints, frontmatter metadata | Clean markdown, not HTML |
| **Tool Access** | MCP servers, search APIs | Protocol-based, not scraping |

**Critical Requirements**:
- Serve markdown to agents automatically (user-agent detection)
- Include `canonical_url`, `last_updated` in frontmatter
- Handle 404s with markdown, not HTML
- Always serve sitemap footer for navigation

### 7. **Prompt Compression Survey Insights**

From the NAACL 2025 survey ([aclanthology.org/2025.naacl-long.368](https://aclanthology.org/2025.naacl-long.368)):

**Hard Prompt Methods** (natural language):
- **SelectiveContext**: Self-information via SpaCy
- **LLMLingua**: Small LM perplexity, up to 20x compression
- **Nano-Capsulator**: Fine-tuned Vicuna-7B for paraphrasing

**Soft Prompt Methods** (continuous vectors):
- **GIST tokens**: 26x compression, modified attention mechanism
- **AutoCompressor**: Recursive compression up to 30,720 tokens
- **ICAE**: 4-16x compression with frozen decoder
- **500xCompressor**: 6-480x compression using KV pairs

**Key Insight**: Soft prompt methods can be seen as a **new synthetic language** for LLMs - more efficient than natural language.

## Advanced Implementation Strategies

### 8. **Context Budget Management**

From mdmin's ContextBudget ([mdmin.dev](https://mdmin.dev/)):

```javascript
const budget = new ContextBudget({
  limit: 128_000,    // model's context window
  reserve: 8_000,    // headroom for output
  keepLastN: 10,     // recent turns always verbatim
})

budget.setSystem('You are a helpful assistant.')
budget.pin('user_id=u_123')      // never dropped
budget.addContext(ragDocument)   // compressed on ingestion
```

**Key Features**:
- **Pins**: Critical facts survive every trim
- **Recent turns**: Always verbatim
- **Old turns**: Rule-compressed first, then dropped oldest-first

### 9. **Research-Backed Design Principles**

From the rate-distortion survey:

1. **Reversibility matters more than scoring**: At same budget, retrieval-backed methods beat eviction/summarization
2. **Query-conditioning pays off**: Offline gisting pays quantifiable penalty vs query-aware compression
3. **Repeated compaction is unmeasured**: Agents compact memory again and again - benchmark this
4. **Attention magnitude is universal signal**: But fails by discarding before query is known
5. **Task-dependent compression**: Tasks with high-entropy answers (multi-hop retrieval) compress poorly

### 10. **Tools Ecosystem**

| Tool | Purpose | Link |
|------|---------|------|
| **llm-docs-builder** | Transform docs for AI, generate llms.txt | [github.com/mensfeld/llm-docs-builder](https://github.com/mensfeld/llm-docs-builder) |
| **doc-trimmer** | Token-efficient context with secret redaction | [npmjs.com/package/doc-trimmer](https://www.npmjs.com/package/doc-trimmer) |
| **markdown-matters** | Extract structure (80%+ fewer tokens) | [github.com/srobinson/markdown-matters](https://github.com/srobinson/markdown-matters) |
| **SMELT** | Query-conditioned retrieval (94-96% savings) | [github.com/TooCas/SMELT](https://github.com/TooCas/SMELT) |
| **Ruler** | Synthesize instructions for multiple agents | [github.com/intellectronica/ruler](https://github.com/intellectronica/ruler) |

## Key Takeaways for `compress-llm-documentation` Skill

1. **Implement hybrid compression**: Summarize → Chunk → Token-trim
2. **Follow AGENTS.md spec**: Minimal, toolchain-first, avoid pink elephant
3. **Use deterministic rules first** (mdcompress Tier-1/2), then LLM rewriting with faithfulness audit
4. **Preserve code blocks** and verbatim material
5. **Implement position-aware budgeting**: Head/tail priority
6. **Add dictionary deduplication** for repeated phrases
7. **Include metadata**: Frontmatter with canonical_url, last_updated
8. **Test with actual LLMs** - compression that works on benchmarks may fail in production
9. **Measure both compression ratio and task F1** jointly
10. **Consider reversibility**: Retrieval-backed > eviction for agent memory

**Most important**: The field is moving from "compress everything" to "preserve decision-relevant bits" - optimize for downstream task utility, not just token count.