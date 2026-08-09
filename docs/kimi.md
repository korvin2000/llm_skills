I'll research current SOTA practices across multiple angles: agent file specs, prompt compression research, and context engineering.

Good foundation. Now researching format-level optimizations (tables vs prose, XML vs Markdown, token costs) and context engineering research.

One more round on niche angles: instruction-vs-example efficacy, tokenizer quirks, and practical compression notation.

# Research Report: Compressing & Optimizing LLM-Facing Markdown Documentation

Knowledge base for a `compress-llm-documentation` skill. Evidence-weighted, opinionated, with sources at the end.

---

## 1. First Principles (why compression works & when it helps)

| Principle | Evidence | Design consequence |
|---|---|---|
| **Attention budget is finite** — every token dilutes signal | Anthropic context-engineering post: n² pairwise attention, "smallest possible set of high-signal tokens" | Deletion > paraphrase > reformat. First pass = cut, not rewrite |
| **Context rot** — recall & instruction-following degrade as context grows; ~7.9% accuracy loss from length alone (Chroma 18-model study) | Every model degraded; 1M models "lie" past ~200K | Short file > long file even if everything in the long file is relevant |
| **Lost in the middle** — U-shaped position bias; positions 5–15 of 20 = "death zone" (-30pp) | Liu et al. 2023, replicated at frontier scale | Put critical rules at top AND repeat hard constraints at bottom; bury nothing important mid-file |
| **Instruction drift** — system-prompt adherence decays as conversation lengthens | Context-rot taxonomy | Concise files survive longer sessions; redundancy ≠ noise if placed at boundaries |
| **Natural language is redundant** (Shannon); low-perplexity tokens carry little info | LLMLingua (EMNLP'23, up to 20× compression), Selective-Context | High-PPL content (numbers, names, flags, negations, examples) = keep; predictable filler = cut |
| **Generic instructions are negative-value** — LLM-generated AGENTS.md *reduce* success in 5/8 settings, add 2.5–3.9 steps/task | 2,500-repo study; ETH Zurich | Compression must preserve *project-specific deltas*, not generic advice |

**Well-formed compact files are measurably better than none**: AGENTS.md presence → −28.6% median runtime, −20% output tokens (arXiv 2601.20404). The failure mode is bloat, not the format.

---

## 2. LLM-Optimized vs Human-Optimized Text

| Dimension | Human-optimized | LLM-optimized |
|---|---|---|
| Redundancy | Repetition aids memory | Repetition costs attention; keep only for hard constraints (boundary repetition) |
| Prose flow | Narrative, transitions | Telegraphic, fragmentary; transitions are pure waste |
| Hedging | "generally", "it's recommended to consider" | Direct imperatives; hedging weakens adherence AND costs tokens |
| Examples | Supplementary illustration | **Primary encoding of rules** — 1 example beats 3 paragraphs (see §3) |
| Visual layout | Whitespace, alignment, boxes | Whitespace is tokens; alignment padding in code blocks/tables = waste |
| Implicit context | Reader infers | State invariant + consequence explicitly once, tersely |
| TOC/navigation | Nice for long docs | **Load-bearing**: LLM greps/partial-reads; TOC is its query plan |
| Politeness | Courtesies | Zero cost to omit; "Please kindly" = pure tax |

---

## 3. Wording & Sentence-Level Rules

### 3.1 Directive style
- **Imperative, verifiable, concrete**: `"Use 2-space indentation"` ✓ vs `"Format code properly"` ✗. Every rule must be machine-checkable-in-principle.
- **One fact per line, one line per fact.** Bullets > paragraphs. Claude Code docs confirm: headers+bullets scanned like a reader would, dense paragraphs lose rules.
- **Litmus test per line** (Red Hat): *"Would removing this line cause a mistake the agent wouldn't otherwise make?"* No → delete.
- **Kill discoverable content**: directory trees, dependency lists, standard README content — the agent can `cat package.json`. Claude Code `/doctor` now auto-trims exactly this class.
- **State the consequence for gotchas**: `"Catch specific exceptions — broad catches mask real bugs"` — the *why* is what makes the rule stick under generalization pressure.

### 3.2 Examples > instructions (strongest single finding)
- NeurIPS'24 (Teach Better or Show Smarter): exemplar selection can eclipse instruction optimization; LLMs "respond better to exemplars from which they can copy behaviors."
- AGENTS.md repo study: "One code example per convention beats three paragraphs" — **the single most important insight** across 2,500 repos.
- Pattern: `// CORRECT` + `// WRONG` pair, minimal, side-by-side. The contrast pair encodes the decision boundary more densely than prose.
- **Pseudocode is a compression format**: encode workflows as 5-line pseudocode instead of 3 paragraphs. LLMs parse code blocks with higher fidelity than prose (trained on massive code corpora; code = unambiguous structure).

```text
on_pr:
  run: lint --fix && test
  if fail: fix, retry ×2
  then: commit "feat: <scope>: <desc>"
```

### 3.3 Negations & constraints
- Prefer positive rules; where negation needed, use explicit `NEVER`/`ALWAYS` markers — capitals act as attention anchors.
- Three-tier boundary pattern (Atlan consensus): `ALWAYS` / `ASK FIRST` / `NEVER` — decision-table format:

| Tier | Rule |
|---|---|
| NEVER | edit `generated/`, commit secrets, touch prod DB from tests |
| ASK | schema migrations, dependency upgrades |
| ALWAYS | run `bun test` before commit |

### 3.4 Information-density heuristics (compression scoring)
Score each sentence/line by what LLMLingua-style analysis would keep:
- **Keep**: proper nouns, exact commands+flags, versions, numbers, paths, negations, decision boundaries, non-obvious invariants, rationale for surprising rules.
- **Cut**: anything the model knows from pretraining (language basics, "write clean code"), anything derivable by reading the repo, synonyms/restatements, examples of the obvious.
- Rule of thumb: **information gain = P(agent error without line) × error cost**. Sort lines by this; truncate the tail.

---

## 4. Format & Structure Optimization

### 4.1 Format selection matrix (empirical)

| Data shape | Best format | Token evidence |
|---|---|---|
| Uniform flat records (≥5 rows) | **Markdown table or TSV/CSV** | TSV −62%, MD-table −54% vs pretty JSON |
| Nested config / trees | **Compact YAML or compact JSON** | YAML best accuracy (Improving Agents); YAML ≈ −23–31% vs pretty JSON; indentation tax makes YAML worse than compact JSON on deep nesting |
| Rules/instructions | **Markdown KV (`key: value`)** | Best accuracy (60.7%, +16pp over CSV) but 2.7× tokens — use for *high-stakes* small blocks |
| Mixed uniform arrays in tools | TOON | −30–60% vs JSON on uniform arrays; **+11–22% worse than compact JSON on nested/non-uniform** — do not default to it |
| Anything | **Avoid XML & pretty JSON** | XML +80% vs MD, worst accuracy; closing tags write every field twice |

⚠️ Nuance (arXiv 2602.05447, 9,649 experiments): format has **no significant aggregate accuracy effect** (p=0.484); model capability dominates. Optimize format for **tokens**, not accuracy — and prefer familiar formats (grep-density and pattern familiarity matter more than novelty).

### 4.2 Markdown micro-costs
- Each `|` and `-|-` separator row in tables = real tokens; tables win only with ≥4–5 rows and ≥3 columns. Below that, KV lines are cheaper.
- Code fences: use ` ```text ` / bare fences; the language tag is 1–2 tokens, usually worth it for parsing fidelity.
- Headings: `##` hierarchy is nearly free and high-value (attention anchoring + grep targets). Prefer more headings, shorter sections.
- Bold = 4 chars per span; bold only trigger words (`NEVER`, exact commands), not sentences.

### 4.3 Tokenizer-aware writing (non-trivial, rarely documented)
- **ASCII > Unicode**: emoji = 2–4 tokens each; CJK ≈ 1.5 tok/char (vs 0.25 for Latin); Cyrillic ~2× English cost. Docs consumed by frontier models should be English-ASCII even if the team is not — BPE merges favor English.
- Prefer `->` over `→` (1–2 tokens vs often 1; both fine, but consistency matters more), avoid box-drawing art, decorative `═══` separators.
- **Word choice by tokenization**: common merged tokens are cheaper — `tokenization` (1 tok) vs rare compounds. Short common words > long rare words, counterintuitively *for tokens*; but rare precise words can be *better for signal*. Optimize: shortest **unambiguous** term, define jargon once in a glossary line.
- Numbers/paths/flags tokenize fine — don't spell them out (`--fix` not "the fix flag").
- Never trust `chars/4` token estimates for non-ASCII (6× underestimate for CJK).

---

## 5. Document Architecture: Progressive Disclosure (the big win)

The dominant strategy across Claude Skills, Claude Code rules, AGENTS.md practice: **never put everything in one file; build an index + on-demand layers.**

### 5.1 Three-layer model
```
Layer 0 (always loaded, system prompt): metadata/index — name, description, routing table
Layer 1 (loaded on trigger): SKILL.md / AGENTS.md core — ≤150–300 lines
Layer 2 (loaded on demand): reference/*.md, examples, scripts (executed, never loaded)
```

### 5.2 Hard limits (empirical ceilings)
| Artifact | Limit | Source |
|---|---|---|
| AGENTS.md | **≤150 lines** (diminishing returns above; +20–23% cost, no gain); 60–300 practitioner range; 32 KiB hard max | 2,500-repo study, Red Hat, Atlan |
| CLAUDE.md | **≤200 lines** ("longer files reduce adherence") | Anthropic docs |
| MEMORY.md | 200 lines / 25 KB loaded; **one line per entry**, detail → topic files | Claude Code auto-memory |
| SKILL.md body | ≤500 lines; split at approach | Anthropic skills docs |

### 5.3 Splitting rules
- **Index/orientation table beats prose links** (agent-optimized, not human-optimized):

```markdown
| Topic | File |
|---|---|
| Setup & commands | README.md |
| Architecture & diagrams | docs/ARCHITECTURE.md |
| API contracts | specs/README.md |
| Testing conventions | .claude/rules/testing.md |
```

- **One level deep only.** Nested references (a.md → b.md → c.md) cause partial reads (`head -100`) and information loss. All reference files link directly from the root index.
- **Reference files >100 lines need a TOC at top** — the LLM uses it as a query plan when partial-reading.
- **Path-scoped loading** (`.claude/rules/` with `paths:` frontmatter): rules load only when matching files are touched — the ideal "compression" is *conditional non-loading*. A compression skill should actively propose this restructure.
- **Scripts over instructions**: a deterministic procedure → bundle as executable script. Executed code costs zero context tokens; described code costs tokens every load.
- **TOC + grep hint**: give the agent a `grep -i "revenue" reference/finance.md` pattern — retrieval instructions are cheaper than content.

### 5.4 Positioning within a file
1. Top: 1-line purpose + hard constraints (primacy zone).
2. Middle: reference/index tables (safe to be "lost" — agent navigates by heading).
3. Bottom: repeat the 1–3 most critical NEVER/ALWAYS rules (recency zone). Deliberate boundary-redundancy is the one justified duplication.

---

## 6. Compression Pipeline (stages for the skill)

```
Stage 0 MEASURE  → token count per section (tiktoken/cl100k or o200k as proxy); baseline
Stage 1 TRIAGE   → classify each line: INVARIANT | COMMAND | CONVENTION | GOTCHA |
                   GENERIC | DERIVABLE | STALE | DUPLICATE
Stage 2 DELETE   → GENERIC (model already knows), DERIVABLE (repo-readable),
                   STALE (verify commands/paths exist!), DUPLICATE (keep one canonical)
Stage 3 CONVERT  → prose→bullets; rules→examples (CORRECT/WRONG pairs);
                   workflows→pseudocode; enumerations→tables; repeated keys→tabular
Stage 4 CONDENSE → telegraphic rewrite: drop articles/fillers/hedges; merge parallel
                   bullets; glossary-compress jargon (define once, abbreviate after)
Stage 5 SPLIT    → if >limit: extract Layer-2 files, build orientation table, add TOCs;
                   propose path-scoped rules / skills for conditional content
Stage 6 POSITION → hard constraints to top+bottom; index tables mid-file
Stage 7 VALIDATE → re-count tokens (report ratio); diff-check every command, path,
                   version, flag against source (compression must be lossless for
                   facts); check contradictions (conflicting rules → arbitrary choice)
```

**Stage 7 is non-negotiable**: compression of *technical* docs must be fact-lossless. Any rewording of a command, flag, version, or path is a bug. Skill should never paraphrase inside backticks.

---

## 7. Non-Trivial / Original Techniques

1. **HTML comments as zero-cost human notes** — Claude Code strips block-level `<!-- -->` from CLAUDE.md before injection. Human maintainers get rationale; the model pays nothing. Compression skill can move "why" prose into comments. (Comments inside code blocks are preserved — don't touch those.)
2. **Notation layer**: define a 3-line legend, then use symbols: `→` implies, `≠` contrast, `∴` therefore, `!` warning, `?` ask-first. After definition, symbols parse reliably and save tokens at scale. Same trick as a math paper's notation section.
3. **Formulae as spec**: `compression_ratio = tokens_after / tokens_before; target ≤ 0.4` — one line replacing a paragraph of policy. LLMs handle inline math fine.
4. **Diff-format for conventions**: instead of describing style, show `- bad_pattern` / `+ good_pattern` unified-diff style. Diff is a native LLM dialect; extremely dense.
5. **Decision tables > if/else prose** for boundary rules (see §3.3).
6. **Glossary compression**: first mention `Authentication Service (auth-svc)`, then `auth-svc` throughout. Classic, underused in agent files.
7. **Negative knowledge is the highest-density content**: "we use X, **not Y**" and "gotcha: Z fails silently" — these lines have the highest P(error without line). In compression, cut positive obvious rules *before* touching a single gotcha. Anthropic's Claude-5 context post agrees: "spend most of the tokens on gotchas."
8. **Single source of truth via symlinks**: `CLAUDE.md → AGENTS.md` symlink; conflicting tool-specific files cause inconsistent behavior. Skill should detect duplicates across `AGENTS.md`/`CLAUDE.md`/`.cursor/rules`/`GEMINI.md` and merge.
9. **Contradiction audit**: two conflicting rules → model picks arbitrarily. Compression must dedupe *semantically*, not just textually — merge overlapping rules into one stricter statement.
10. **Prompt caching interaction** (Claude): CLAUDE.md is cache-read after first hit (~5 min TTL); edits invalidate cache. Implication: batch all doc edits; stable prefix ordering matters. Size still costs attention even when it doesn't cost money — **optimize for attention, not price**.
11. **Anti-pattern: LLM-generated agent files.** Do not let the skill *invent* content — it must only delete/restructure what humans wrote, flagging gaps as comments (`<!-- GAP: no test command -->`) rather than filling them with generic prose.
12. **LLMLingua-style budget allocation, manually**: don't compress uniformly — allocate budget by section criticality: gotchas/constraints 0% compression, conventions 50%, background/overview 80%. Uniform summarization is the failure mode of naive "compress this doc" prompts.
13. **Multilingual docs**: if docs exist in non-English, compressing *and translating to English* often nets −30–60% tokens purely from tokenizer economics — worth proposing in the skill (with a human-review flag).

---

## 8. Suggested `compress-llm-documentation` SKILL.md Skeleton

```markdown
---
name: compress-llm-documentation
description: Compresses and restructures LLM-facing Markdown docs (AGENTS.md,
  CLAUDE.md, memory files, skills) for token efficiency and adherence without
  factual loss. Use when optimizing agent instructions, memory files, or
  reducing context bloat.
---

# Compress LLM Documentation

## Invariants (NEVER violate)
- Never alter text inside backticks/code blocks (commands, flags, paths, versions)
- Never invent content; flag gaps as <!-- GAP: ... -->
- Never compress gotchas/NEVER-rules beyond 0%; never merge two constraints into ambiguity

## Pipeline
1. Measure: token count per section → baseline table
2. Triage: tag lines INVARIANT/COMMAND/CONVENTION/GOTCHA/GENERIC/DERIVABLE/STALE/DUP
3. Delete: GENERIC, DERIVABLE, STALE (verify!), DUP
4. Convert: prose→bullets, rules→CORRECT/WRONG examples, workflows→pseudocode, enums→tables
5. Condense: telegraphic rewrite; see reference/style-rules.md
6. Split: if >150 lines (AGENTS.md) / >200 (CLAUDE.md) → extract + orientation table;
   see reference/splitting.md
7. Position: hard constraints top+bottom
8. Validate: diff all facts vs source; contradiction audit; report token ratio

## References
- Wording rules & litmus tests: reference/style-rules.md
- Format selection matrix: reference/formats.md
- Splitting/progressive disclosure patterns: reference/splitting.md
- Worked example (before/after): reference/example.md
```

---

## 9. Key Resources

| Resource | Why |
|---|---|
| [Anthropic — Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) | Attention budget, just-in-time context, compaction, note-taking — the canonical framing |
| [Claude — New rules of context engineering (Claude 5 gen)](https://claude.com/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models) | "Spend tokens on gotchas"; latest vendor guidance |
| [Claude Code memory docs](https://code.claude.com/docs/en/memory.md) | 200-line rule, path-scoped rules, HTML-comment stripping, /doctor trims |
| [Agent Skills best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) | Progressive disclosure, one-level-deep refs, TOC rule, 500-line SKILL.md |
| [Red Hat — AGENTS.md standardization (07/2026)](https://developers.redhat.com/articles/2026/07/27/standardize-project-context-agentsmd-and-agent-skills) | Orientation tables, 150-line limit, litmus test |
| [AGENTS.md Best Practices 2026 (2,500-repo analysis)](https://www.betterclaw.io/blog/agents-md-best-practices) | Examples>prose, anti-autogen evidence, section ordering |
| [arXiv 2601.20404 — AGENTS.md efficiency impact](https://arxiv.org/html/2601.20404) | Quantified: −28.6% runtime, −20% output tokens |
| [Improving Agents — format benchmarks](https://www.improvingagents.com/blog/best-nested-data-format/) + [table formats](https://www.improvingagents.com/blog/best-input-data-format-for-llms/) | Markdown-KV accuracy, MD token efficiency, XML worst |
| [Token cost across 9 formats (measured)](https://jangwook.net/en/blog/en/llm-token-cost-data-format-experiment/) | TSV −62% vs pretty JSON; nested flips to compact JSON |
| [TOON format + honest counter-benchmark](https://github.com/toon-format/toon) / [Karvics benchmark](https://karvics.com/toon-benchmark) | Where TOON wins (uniform arrays) and loses (nested) |
| [arXiv 2602.05447 — Structured context engineering at scale](https://arxiv.org/abs/2602.05447v2) | Format ≠ accuracy (p=0.484); model capability dominates; domain-partitioned schemas scale to 10K tables |
| [LLMLingua (EMNLP'23)](https://aclanthology.org/2023.emnlp-main.825.pdf) | Budget allocation, PPL-based token retention — theory for §3.4/§7.12 |
| [Teach Better or Show Smarter (NeurIPS'24)](https://papers.nips.cc/paper_files/paper/2024/file/6b031defd145b02bed031093d8797bb3-Paper-Conference.pdf) | Exemplars > instructions — basis for example-first encoding |
| [Chroma context-rot study summary](https://particula.tech/blog/chroma-context-rot-long-context-degradation) | Length-only floor, position death zone |
| [Lost in the Middle explainer](https://zeroentropy.dev/concepts/context-rot/) | Boundary placement strategy |

---

## 10. My Opinionated TL;DR (skill design priorities)

1. **80% of value = Stage 1–2 (delete)**, not clever rewriting. Most agent files contain 40–60% generic/derivable filler.
2. **Progressive disclosure beats textual compression** — moving 300 lines into indexed reference files saves more attention than any paraphrase, and is fact-lossless by construction.
3. **Examples are the densest rule-encoding** — the skill's most opinionated transformation should be prose-rule → CORRECT/WRONG pair.
4. **Fact-immutability is the trust boundary**: never rewrite inside backticks; validate by diff. A compression skill that silently changes a flag is worse than useless.
5. Skip TOON/XML-novelties for *documentation*; use them only for *data payloads* in tool outputs. Familiar formats (Markdown, YAML, pseudocode, diffs) win on model familiarity.
6. Measure tokens, report ratios, but optimize for **adherence** (rot, position, contradictions) — cost is the secondary metric.