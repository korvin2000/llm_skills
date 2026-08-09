Researching current best practices on LLM-oriented docs, agents.md patterns, and context-compression techniques.

Fetching the highest-signal sources for concrete rules and patterns.

Pulling additional compression skill specs and dual-audience doc patterns.

# Compress-LLM-Documentation — Research Basis

## Core Thesis

**LLM docs ≠ human docs.** Goal: minimize high-signal tokens that drive correct behavior. Every extra token dilutes attention (context rot). AGENTS.md is always-on context → ruthlessly short. Skills use progressive disclosure.

Litmus (Red Hat): *“Would removing this line cause a mistake the agent wouldn’t otherwise make?”* If no → delete.

---

## 1. Architecture: What Goes Where

| Layer | File | Budget | Content |
|-------|------|--------|---------|
| Always-on | `AGENTS.md` | **30–150 lines** | Commands, invariants, boundaries, orientation index |
| On-demand | Skills (`SKILL.md`) | **<500 lines / <5k tokens** | Task procedures |
| Deep ref | `references/`, `docs/*.md` | Unlimited | Architecture, edge cases, examples |
| Memory | memory/notes md | Compressed TLDR | Session facts, prefs, decisions |

**Progressive disclosure (Agent Skills spec):**
1. Metadata only (~50–100 tok/skill): `name` + `description`
2. Full SKILL.md when activated
3. `scripts/`, `references/`, `assets/` only when referenced

**Rule:** Root AGENTS.md = **index + invariants**, not dump. Nested AGENTS.md per package (OpenAI monorepo: 88 files). Closest wins.

---

## 2. LLM-Optimized Writing (vs Human)

| Dimension | Human docs | LLM docs |
|-----------|------------|----------|
| Tone | Narrative, motivation, soft hedges | Imperative, fragments OK |
| Ambiguity | Tolerated (diagrams, tribal knowledge) | Fatal — resolve scope, units, “we” |
| Structure | Skimmable prose + visuals | Tables, bullets, atomic claims |
| Examples | Many, pedagogical | 1 canonical + 1 edge; show not tell |
| Freshness | Visual cues | Explicit frontmatter: `verified:`, `owner:`, `canonical:` |
| Density | Breathable whitespace | High density; blank lines cost tokens too |
| Boundaries | Soft norms | ALWAYS / ASK / NEVER tiers |

**Failure modes unique to agents** (tianpan):
- Stale doc as confident source
- Synthesis across tiers (draft vs canonical)
- Scope collapse (“we use Postgres” → whole company)
- Silent corpus poisoning

**Frontmatter template (agent-critical docs):**
```yaml
---
canonical: true
scope: auth-service only
verified: 2026-08-01
owner: platform
replaces: null
---
```

Deprecated docs: structured tombstone + forward pointer, not a banner humans skip.

---

## 3. Compression Modes

### 3.1 Lossless (safe first pass, ~20–40%)
- Collapse blank lines → 1; strip trailing space
- Remove HR, decorative MD, HTML comments, ToC that duplicates headings
- Normalize bullets; flatten useless nesting
- Merge empty adjacent headers
- Drop redundant emphasis (bold+CAPS+!)

### 3.2 Lossy semantic (~40–70%, needs review)
Principles (markdown-compressor / tldr-compress):

| # | Rule | Example |
|---|------|---------|
| 1 | Imperative > descriptive | `Validate input` not `The system should validate…` |
| 2 | One expression per concept | Deduplicate cross-section |
| 3 | Table > prose | Multi-attribute lists |
| 4 | Inline > nested bullets | `gzip(level=6, min=1KB)` |
| 5 | Delete implied knowledge | No REST/JSON tutorials |
| 6 | Merge sections >50% overlap | |
| 7 | Fragments OK | Drop articles, hedges, transitions |
| 8 | Short synonyms | use/fix/run not utilize/implement |

**Remove always:** motivational filler, hedging, restatements, “now that we covered X”, pleasantries, excessive examples.

**Never remove:** numbers/thresholds, NEVER/ALWAYS rules, paths/tools/endpoints, conditionals, output formats, edge cases, cross-refs, YAML frontmatter, fenced code (byte-exact).

### 3.3 Compressor–Reviewer loop (production pattern)
```
for section in sections:
  compressed = compressor.aggressive(section)
  loss = reviewer.diff(original, compressed)  # rules, numbers, edges, refs
  if loss: reject or repair
  user_approve(diff)  # or --auto after trust
```
Reviewer checks: lost prohibitions, dropped thresholds, over-generalization, broken links.

### 3.4 Token estimate
`tokens ≈ words * 1.3` (relative OK; absolute varies by tokenizer).

---

## 4. Structural Optimization (highest leverage)

### Index pattern (AGENTS.md skeleton)
```markdown
# AGENTS

## Commands
| Action | Cmd |
|--------|-----|
| test | `pnpm test` |
| lint | `pnpm lint --fix` |
| typecheck | `pnpm tsc -b` |

## Stack
Node 22, TS 5.6, pnpm, Vitest, Zod 3

## Layout
- `src/` read+write
- `tests/` write tests only
- `vendor/` never touch

## Boundaries
- ALWAYS: run tests before commit; conventional commits
- ASK: schema, deps, CI
- NEVER: secrets, `vendor/`, force-push main

## Invariants
- Catch specific exceptions only
- Auth module ↛ billing imports
- bcrypt cost ≥ 12; session TTL 30m

## Docs index
| Topic | Path |
|-------|------|
| Architecture | `docs/ARCHITECTURE.md` |
| API contracts | `specs/` |
| Deploy | `docs/RUNBOOK.md` |
```

Commands **early**. Code example > style essay. Six core areas: commands, testing, structure, style, git, boundaries (GitHub 2500-repo study).

### Split large docs
- Vertical: by domain → nested AGENTS.md / skills
- Horizontal: SKILL.md core + `references/edge-cases.md`
- Keep refs **one level deep** from SKILL.md
- Orientation table > long prose pointers

### Self-contained chunks
Each section must stand alone if retrieved alone: name product/scope in first line; no dangling pronouns; code blocks copy-pasteable (imports included).

---

## 5. Non-Trivial / High-Density Techniques

### A. Pseudo-code / procedure DSL
Prose procedures → numbered imperative steps or pseudo-code. Validated: pseudo-code prompts beat NL on many tasks (variable tracking).

```text
PRE: clean tree, on feature branch
1. lint --fix
2. test
3. IF fail → STOP, report
4. commit conventional
5. push -u
```

### B. Decision tables / matrices
```text
| Cond | Action |
|------|--------|
| touches payments | STOP, ask human |
| test fail | fix or abort; never delete test |
| new dep | ASK |
```

### C. Symbolic / telegraph compression (advanced, measure first)
- **Telegraph English** (~50% tokens, ~99% key-fact retention): atomic lines + ~40 relational symbols (`→` cause, `⇒` imply, `∵` because, `∧∨¬`, tags `NOW:`, `CTX:`)
- **MetaGlyph**: math ops models already know (`∈`, `¬`, `⇒`) as instruction shortcuts
- Cap symbol density; self-verify with quality checklist
- **Opinion:** useful for memory/compaction layers; risky for shared team AGENTS.md unless all consumers tested. Prefer tables + imperatives for portability.

### D. Anti-examples (cheap, high signal)
```text
// ✅ fetchUserById(id): guard empty, typed return
// ❌ get(x): no guard, untyped
```
One good/bad pair > paragraphs of style rules.

### E. Formula / constraint encoding
```text
retry: exp_backoff(base=200ms, cap=5s, n≤6)
auth: bcrypt(cost≥12); session_ttl=30m
```

### F. Links as handles (JIT context)
Don’t inline architecture. Point:
`arch → docs/ARCHITECTURE.md#auth`
Agent loads on demand (Anthropic “just-in-time” / progressive disclosure). Metadata in path/name is free signal (`test_utils.py` in `tests/` vs `src/`).

### G. Images
For LLM-only docs: **replace diagrams with structured text** (mermaid source, adjacency lists, sequence as numbered steps). Alt text if image must stay. Agents don’t “see” stale screenshots as humans do.

### H. Memory tiering (memorypack-style)
```
T0 overview ~200 tok
T1 topic summaries ~100 tok each
T2 key facts (proper nouns, numbers, ALWAYS/NEVER) ≤10/cluster
```
Raw files stay; summaries are retrieval layer.

### I. Compaction policy (session, not file)
Preserve: decisions, unresolved bugs, invariants, paths touched.  
Drop: raw tool dumps, redundant turns.  
Anthropic: summary + last N files. Tool-result clearing = safest light compaction.

### J. Dual corpus discipline
- Human README can stay narrative
- Agent surface: AGENTS.md + skills + `llms.txt` index
- Or single source with machine blocks (scope/canonical frontmatter) that humans skim

---

## 6. Skill Spec Blueprint: `compress-llm-documentation`

```yaml
---
name: compress-llm-documentation
description: >
  Compress/optimize LLM-facing Markdown (AGENTS.md, CLAUDE.md, SKILL.md,
  memory, architecture notes). Use when user says compress, shrink tokens,
  densify agent docs, tldr memory, optimize context. Lossless + lossy modes
  with section review.
---
```

### Workflow stages
1. **Classify target:** always-on | skill | reference | memory  
2. **Pre-analyze:** heading tree, tok/section, redundancy map, oversized (>~500 tok)  
3. **Lossless pass** (always)  
4. **Lossy rewrite** per section (imperative, tables, dedupe, drop implied)  
5. **Reviewer gate:** checklist of never-drop items  
6. **Structural refactor if needed:** extract to references/, build orientation table, nest AGENTS  
7. **Validate:** frontmatter intact; code fences exact; links resolve; budget (AGENTS <150 lines; SKILL <500)  
8. **Report:** before/after tokens, % reduction, sections changed, residual risks  
9. **Backup:** `*.original.md` before overwrite  

### Reviewer checklist (must pass)
- [ ] All NEVER/ALWAYS/ASK preserved  
- [ ] All numeric constraints preserved  
- [ ] All paths/commands/env vars exact  
- [ ] Conditionals complete  
- [ ] Output schemas intact  
- [ ] Cross-refs not broken  
- [ ] No invented content  
- [ ] Code blocks byte-identical  

### Anti-goals
- Don’t teach generic programming the model knows  
- Don’t keep auto-generated boilerplate (ETH Zurich: generic AGENTS.md can **hurt**)  
- Don’t compress human marketing READMEs the same way without a dual-audience policy  
- Don’t deep-nest reference chains  

---

## 7. Opinionated Defaults (practical)

1. **Always-on budget is sacred** — move detail out; index in.  
2. **Failure-driven growth** — start minimal; add a line only when agent errs twice.  
3. **Tables + imperatives + 1 example** beat essays.  
4. **Canonical vs draft** must be machine-visible.  
5. **Measure** compression with a tiny eval set (3–5 tasks agent must still pass).  
6. **Telegraph/symbols** for private memory compaction; plain dense MD for shared repos.  
7. **Skills > bloated AGENTS** for situational knowledge.  
8. **Write as if each chunk is the only page retrieved.**

---

## 8. Key Resources

| Resource | Why |
|----------|-----|
| [agents.md](https://agents.md/) | Standard always-on agent context |
| [GitHub: great agents.md (2500 repos)](https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/) | Empirical patterns |
| [Red Hat: AGENTS.md + Skills](https://developers.redhat.com/articles/2026/07/27/standardize-project-context-agentsmd-and-agent-skills) | Budget, index table, progressive disclosure |
| [Anthropic: Effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) | Attention budget, compaction, JIT, note-taking |
| [Agent Skills spec](https://agentskills.io/specification) | SKILL.md format, 3-tier load |
| [markdown-compressor SKILL](https://github.com/oborchers/fractional-cto/blob/main/markdown-compressor/skills/markdown-compression/SKILL.md) | Lossless/lossy rules + reviewer loop |
| [tldr-compress SKILL](https://github.com/0p9b/tldr/blob/main/skills/tldr-compress/SKILL.md) | Article/hedge stripping, preserve code exact |
| [Docs agents vs humans](https://tianpan.co/blog/2026-04-27-docs-for-agents-vs-humans-dual-audience) | Dual audience, staleness, scope |
| [Telegraph English (arXiv)](https://arxiv.org/html/2605.04426) | Symbolic atomic-line compression |
| [MetaGlyph](https://arxiv.org/html/2601.07354v1) | Math-symbol instruction compression |
| [PseudoLangs overview](https://promptengineering.org/introduction-to-pseudolangs/) | Pseudo-code vs minify vs symbols |
| [Writing for humans + models (ReadMe)](https://readme.com/resources/writing-for-humans-and-models) | Exact IDs, units, constraints up front |
| [AI-readable docs / llms.txt](https://doccupine.com/blog/ai-readable-documentation) | llms.txt, self-contained pages |

---

## 9. Minimal “skill body” pseudo-spec you can implement

```text
INPUT: path, mode∈{lossless,lossy,restructure}, auto∈{bool}

ANALYZE → section_table(tokens, flags:redundant|oversized|generic)

IF restructure:
  extract always_on_invariants
  move bulk → references/ or docs/
  write orientation_table into AGENTS/SKILL

FOR each section:
  out = lossless(section)
  IF lossy: out = rewrite_dense(out)  # rules §3.2
  assert preserve_set(original, out)
  IF NOT auto: await approve(diff)
  write incremental

REPORT metrics + residual_risks
BACKUP *.original.md
```

**rewrite_dense heuristics:** strip detokenizer-waste (articles, hedges); prefer `|col|` tables; fold nested lists; single best example; project-specific only; keep heading skeleton.

---

This is sufficient to implement `compress-llm-documentation` as a skill: dual-mode compression, section gates, AGENTS/Skills budget rules, progressive-disclosure refactor, and optional telegraph layer for memory. Highest ROI order: **restructure to index → lossless → lossy with reviewer → failure-driven line adds**.