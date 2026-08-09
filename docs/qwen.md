# Research Report: Compressing & Optimizing LLM-Facing Markdown (basis for a `compress-llm-documentation` skill)

Sources: Gloaguen et al. 2026 (arXiv:2602.11988, ETH), Anthropic Skills/CLAUDE.md docs, asdlc.io AGENTS.md spec, BulkMD token census 2026, arXiv:2607.19257 (Prompt Design at Scale), arXiv:2606.19857 (BabelTele), LLMLingua/-2, Lost-in-the-Middle (arXiv:2307.03172), Chroma context-rot, CodeKunda system-prompt optimization study, tellian.io 2026 survey, format benchmarks (improvingagents, thoeltig, yaml-vs-md-benchmark), memory-bank/architecture writeups.

---

## 1. Core thesis (evidence-backed, my framing)

**Compression of LLM docs is ~80% deletion, ~15% restructuring, ~5% rewriting.** Measured data: in a controlled human-vs-machine format study, 81% of savings came from removing prose/filler/whitespace, 16% from markup, only 3% from abbreviations. Over-compression is real and harmful: agents *faithfully follow* every extra rule, which raises reasoning tokens +20–22% and steps +2.5–3.9, and LLM-generated verbose context files **reduce** task success in 5/8 settings.

| Finding | Number | Implication |
|---|---|---|
| LLM-generated AGENTS.md vs none | −success, +20–23% cost | Never ship `/init` output as-is |
| Human-written minimal context file | +4% success only | Only minimal, precise content pays |
| AGENTS.md sweet spot | <150 lines (30–50 for small repos) | Hard budget in your skill |
| SKILL.md body | <500 lines | Progressive disclosure above that |
| MEMORY.md load limit (Claude Code) | 200 lines / 25 KB | Index-style memory files |
| Telegraphic rewrite of instructions | −15–21% tokens (not "−75%") | Realistic rewrite target |
| Aggressive model-native compression (BabelTele) | 28% length, 99.5% semantic fidelity in QA | Viable only for machine-only blobs |
| Over-compression side effect | reader CoT tokens rise | Moderate ratio ≈ optimal |
| Instruction adherence | degrades with rule count & length | Fewer rules = better compliance |

**Litmus test every line must pass:** *"If deleted, would the agent make a mistake it wouldn't otherwise make?"* No → delete.

---

## 2. Token economics (what costs what)

cl100k/o200k/Claude tokenizers agree within a few %. Tokens per 1 KB:

| Content | tok/KB | Content | tok/KB |
|---|---|---|---|
| Bullet list (MD) | ~270 | JSON minified | ~335 |
| English prose (MD) | ~280 | YAML | ~335 |
| CSV | ~295 | JSON pretty | ~400 |
| MD pipe table | ~315 | Python code | ~420 |
| Prose in HTML | ~325 | TypeScript code | ~460 |
| HTML `<ul>` | ~350 | HTML table | ~625 |

Rules derived:
- **MD beats HTML everywhere** (10–50%); HTML→MD table conversion alone = 30–40% shrink.
- **Code is the most expensive content per byte.** Don't inline implementations; inline *types + signatures + behavior notes* (types are the highest value-per-token code artifact).
- Format overhead vs plain text: MD 1.258×, prose 1.221×, MD tables 1.367× (arXiv:2607.19257). Tables cost tokens but raise adherence — keep them small and only for machine-parsed registries (commands, endpoints).
- **Language matters multiplicatively:** English ≈ 4 chars/token (densest). Russian ≈ ×2, Slavic languages worst end; Chinese saves nothing (myth, tokenizer fragmentation eats ideograph density). → **Instruction core always in English.**
- Emojis/flags/box-drawing: 1–10+ tokens each, zero signal. Program symbols (`=`, `→`, `|`, `:=`, `?:`) are cheap and in-distribution.
- Bold/italic pairs, backtick pairs, redundant fences add ~tokens for nothing: keep `##` headers (navigation), backticks only on real identifiers, drop emphasis markup.

---

## 3. What an LLM-optimized document looks like (vs human doc)

| Dimension | Human doc | LLM doc |
|---|---|---|
| Unit | paragraph + narrative glue | one atomic rule/fact per bullet line |
| Completeness | exhaustive, educational | only delta vs model's prior knowledge ("things that would surprise a framework expert") |
| Negations | "don't use X" | positive form "use Y" (negation primes the forbidden token — Pink Elephant / context anchoring) |
| Verifiability | vague ("format code properly") | falsifiable ("2-space indent"; "run `npm test` before commit") |
| Detail placement | inline | pointer + one-line summary (lazy load) |
| Layout | any | critical rules at start AND end (U-shaped attention: lost-in-the-middle −20–30pp) |
| Duplication | tolerated | forbidden: two copies drift; contradictions make model pick arbitrarily |
| Discoverable facts | restated | deleted (agent reads README/config itself; "the tool is the constraint" — linter rule ≠ doc rule) |
| Logic description | prose | decision table / pseudocode / `if X → Y` arrows |

---

## 4. Compression pipeline (recommended architecture for the skill)

Run as ordered stages; each stage has higher ROI than the next:

```text
STAGE 0 AUDIT     count tokens with real tokenizer (never chars)
                  classify spans: prose|table|code|list|yaml|json|html
                  detect: duplicates, stale/contradictory rules, toolchain-covered rules
STAGE 1 DELETE    litmus test per line; drop:
                  - model-already-knows (generic framework facts)
                  - repo-discoverable (dir layout, README copies)
                  - linter/formatter/type-enforced ("tool is the constraint")
                  - aspirational/unenforced rules
                  - history narration (git is the changelog)
STAGE 2 RELOCATE  3-tier progressive disclosure:
                  T1 always-loaded core (<150 lines: mission 2-4 sent.,
                     commands table, NEVER/ASK/ALWAYS, gotchas)
                  T2 indexed (path + 1-line summary, one level deep only)
                  T3 on-demand (URL/path pointers, ~10 tok each vs 5000 inline)
                  split >500-line bodies; TOC for >100-line ref files
STAGE 3 REWRITE   prose→telegraphic bullets (cut glue: articles, politeness,
                  transitions); paragraphs→lists; negations→positive;
                  keep verbatim: identifiers, paths, commands, flags, values,
                  error strings (these are retrieval keys — never paraphrase)
                  logic → pseudocode/decision tables/arrows
STAGE 4 FORMAT    HTML→MD (tables 2x, lists ~25%); pretty JSON→minified or YAML;
                  nested data→YAML; language-tag all code fences;
                  drop bold/italic/decorative chars; consistent separators only
STAGE 5 LAYOUT    hard constraints + most-violated rules at file top AND bottom;
                  orientation table near top; stable section order (cache-friendly);
                  output-format spec at tail
STAGE 6 VERIFY    token delta report; fact-inventory diff (every command/path/
                  value/number must survive verbatim); round-trip test:
                  LLM answers golden questions from compressed doc only;
                  optional A/B: task success before vs after
```

### Stage-1 deletion checklist (the money stage)
- Constraint enforceable by linter/CI/types/hooks? → delete from doc, keep `Lint: pnpm lint (see biome.json)` pointer.
- Restates README/architecture/code? → delete (redundancy is the measured failure mode).
- Describes standard framework dirs? → delete.
- Rule for a mistake made once? → delete; add rule only **after the agent errs twice**.
- Two files say the same thing? → keep one, replace with pointer.
- Contains "rejected alternative" knowledge ("evaluated X, rejected because…")? → **keep** — this is the single most valuable memory content (git can't record non-decisions).

### Stage-2 relocation pattern (AGENTS.md as index, not dump)
```markdown
## Toolchain
| Action | Command | Authority |
|---|---|---|
| Test | `pnpm test` | vitest, see vitest.config.ts |
| Lint | `pnpm lint --fix` | Biome config is source of truth |

## Read when needed
| Topic | File |
|---|---|
| Architecture | docs/ARCHITECTURE.md |
| API contracts | specs/README.md |
| Deploy gotchas | docs/DEPLOY.md |

## NEVER
- commit .env / secrets
- add deps without asking
## ASK: migrations, deletions
## ALWAYS: plan before code; explicit error handling
```
Rules: links **one level deep only** (nested refs get `head -100` previewed and half-read); every pointer states *when* to read it; keep static prefix stable for prompt-caching.

---

## 5. Rewrite rules (telegraphic/caveman, done right)

Measured savings 15–21% on instructions; ~39% session-wide with caching. CaveAgent: −28% total tokens with *rising* success rate.

| Rule | Example |
|---|---|
| Cut glue words, keep substance | "Please ensure that all API routes are protected by auth middleware" → `API routes: auth middleware required` |
| One fact per line, nominal style | no "Additionally/Furthermore/Note that" |
| In-distribution shorthand OK | `→`, `:=`, `|`, `?`, `s/`, regex, CLI flags, `grep -i "x" file.md` |
| **Do NOT invent custom grammar** | `KEY=val;val;val` notation measured net-negative: more delimiter tokens, model must learn your grammar before using content; conflicts with BPE training distribution → extra inference compute |
| Never abbreviate identifiers | `s/` for `service/` saves ~1 tok, breaks grep and retrieval; abbreviations = only 3% of savings |
| Compress description, not interface | signatures, event names, error codes, limits stay verbatim |
| Replace prohibition with positive + sample | "don't use tRPC" → `RPC layer: gRPC (see proto/)` |
| Sample beats spec | one concrete input→output example > 10 lines of description |

Pseudocode/logic compression examples that work well:
```text
retry: 3x, backoff 2^n*100ms, jitter; abort on 4xx
auth: token→validate→(ok? next : 401)
if rate>100/s → 429 + Retry-After
```
Math-ish/logic-ish notation is dense AND in-distribution — one of the few genuinely non-trivial wins.

---

## 6. Format-selection evidence (contradictory benchmarks → practical verdict)

Benchmarks disagree because tasks differ:

| Benchmark | Verdict |
|---|---|
| improvingagents (11 formats, retrieval QA) | Markdown-KV best accuracy (60.7%) but 2.7× CSV tokens; CSV cheapest, worst accuracy |
| thoeltig (structural/retrieval) | CSV best for dense mandatory data; YAML highest accuracy; MD tables collapsed to 24.7% |
| yaml-vs-md (Claude Haiku) | TOON −62% vs JSON, no accuracy loss |
| arXiv:2602.05447 (9,649 runs, 11 models) | **No significant aggregate format effect (p=0.484)**; model capability dominates; novel formats can *increase* runtime tokens via grep-output density |

My verdict for the skill: **default = terse Markdown (prose + bullets + small pipe tables); YAML for nested config-like data; JSON only minified and only where code parses it; never convert to exotic formats (TOON etc.) unless measured on the target model/task.** Format choice is secondary to *consistency*: mixing separators section-to-section measurably shifts results.

---

## 7. Attention-aware layout (quality lever, free)

- U-shaped attention: primacy + recency strong, middle weak (−20–30pp). → hardest constraints + current focus at **top and bottom**.
- Context rot: similar-but-irrelevant chunks actively degrade; effective window ≪ spec (half of models fail at 32K on NoLiMa/RULER). → "dump everything just in case" is anti-optimization.
- Explicit index pointers work ("see §Toolchain", "block #2"); vague "pay attention to…" doesn't.
- Few-shot/example position moves accuracy up to 50pp — place canonical examples near the task/question.
- In huge harness contexts, restate 1–3 critical rules in a tail "REMINDER" block.

---

## 8. Memory-file specifics (markdown memory, MEMORY.md, memory banks)

- Structure = **index + topic files**: MEMORY.md ≤200 lines/25KB, one line per entry, details offloaded to `memory/topic-*.md` (Claude Code enforces exactly this).
- Write-time compression > read-time: extract atomic facts when writing, dedup + contradiction-resolve then ("supersede + provenance"), so every read is cheap and precise.
- Recency tiering: recent = full fidelity; >7d = one-line summary; >30d = key facts only.
- Discipline rules that prevent bloat: *lessons live once* (single copy + pointer), *done means gone* (active context ≤80 lines, commit is the record), size cap = staleness audit trigger ("would this help or overwhelm a search?").
- Keep: decisions, rejected alternatives, gotchas, silent invariants, non-obvious conventions. Drop: process narration, per-session diff re-telling, greetings/filler.
- Compaction event = extraction event: before summarizing old turns away, mine durable facts out of them.

---

## 9. Skills-file specifics (SKILL.md)

- Frontmatter `description` is the only always-loaded part (~100 words): third person, contains WHAT + WHEN + trigger keywords. All triggering info lives there, not in the body.
- Body <500 lines; reference files one level deep; TOC in any ref file >100 lines; domain-partitioned refs so only relevant file loads.
- Scripts > prose: if a procedure is deterministic, bundle `scripts/x.py` — **executed, not loaded** (0 context tokens). This is the ultimate compression: move knowledge into code.
- Assume the model is competent: don't teach general programming; only deltas.

---

## 10. Non-trivial techniques & tricks (brainstormed, with my assessment)

| Technique | Mechanism | Verdict |
|---|---|---|
| BabelTele-style model-native blobs (symbols, cross-lingual lexemes, arrows) | 27.9% length, 99.5% semantic fidelity; transfers zero-shot across models; in agent memory beats plain summarization (96.5% vs 94.2% retention) | Use ONLY for machine-only memory/inter-agent blobs. Compressor↔reader pair matters (GPT/Claude-compressed most portable). Never for human-reviewed docs. Never safety-critical |
| Space–time tradeoff awareness | stronger compression → more reader CoT tokens | Target moderate compression (2–4×). >5× on instructions usually net-negative |
| Executable constraints | move rule from doc to linter/script/test | Best compression = 100% (rule leaves context entirely) |
| Types-as-specs | `type ToolStatus = "idle"\|"running"\|"failed"` | Most token-efficient code artifact; encodes whole behavioral contract |
| Endpoint/API maps | only used endpoints + auth + rate limit + error signatures | ~10× cheaper than pasted docs, higher utility |
| URL-as-pointer with trigger note | `Status page: status.project.com (check if API calls fail)` | ~10 tokens; staleness-proof |
| Orientation/context map in YAML | compact repo map | Include only dirs that *deviate* from framework convention |
| `grep` recipes inside docs | `grep -i "revenue" reference/finance.md` | Teaches cheap retrieval instead of loading content |
| Stable doc prefix | cache-friendly ordering | With prompt caching, token price amortizes; then prioritize adherence over last-token savings |
| Separators as API | consistent `##`/XML boundaries across whole doc | Single-char separator changes shift evals by tens of %; never mix styles |
| Images/diagram links | — | **Anti-pattern** for text-reading coding agents: mermaid/ASCII art costs tokens, image links aren't fetched. Replace with 2–3 sentence architecture line |
| Gist/latent compression (26×) | requires finetune/activation access | Not applicable to black-box docs; conceptual only |
| LLMLingua-2 as optional skill step | token-level pruning, instruction-aware budgets | Compress context/demos hard, instructions/questions lightly; sentence-level dropout at high ratios. Optional stage 3.5 |

---

## 11. Draft skeleton for the `compress-llm-documentation` skill

```markdown
---
name: compress-llm-documentation
description: Compresses and rewrites LLM-facing markdown (AGENTS.md, CLAUDE.md,
  memory files, SKILL.md, docs) into high-signal, token-cheap context. Use when
  asked to shrink, optimize, deduplicate or restructure agent instructions,
  memory notes or documentation for LLM consumption.
---

# Compress LLM Documentation

## Pipeline (run in order, report token delta per stage)
1. AUDIT: token count (real tokenizer), span types, dup/stale/tool-covered rules
2. DELETE: litmus test; drop discoverable/enforced/aspirational/redundant
3. RELOCATE: T1 core (<150 lines) | T2 one-level pointers | T3 URL/path only
4. REWRITE: telegraphic bullets; positive rules; verbatim identifiers;
   logic→pseudocode/tables/arrows
5. FORMAT: HTML→MD, JSON→minified/YAML, tag fences, drop emphasis/emoji
6. LAYOUT: critical rules top+bottom, orientation table, stable order
7. VERIFY: fact inventory diff + golden-question round-trip + token report

## Hard rules
- Keep verbatim: commands, paths, identifiers, flags, values, error strings
- Never invent custom notation; only in-distribution symbols
- No negation rules; no restating linters; no dir maps of standard layouts
- English instruction core
- Budgets: AGENTS.md <150L; SKILL.md body <500L; memory index <200L/25KB
- Target 2–4× compression; beyond that run round-trip test first

## Verification output
before/after tokens, % removed, fact-inventory pass/fail, list of pointer-ized items
```

## 12. My top opinions (prioritized)

1. **Delete > rewrite > reformat.** If your skill does only one thing well, make it the Stage-1 deletion with the litmus test.
2. **A doc is a cache hierarchy, not a document.** Design for progressive disclosure; the best token is the one never loaded.
3. **Verbatim preservation of identifiers is the most common compression failure** — paraphrased paths/commands silently break agent retrieval. Build it in as an invariant check.
4. **Keep terse human-readable Markdown as default.** Custom/compressed notations usually lose net (parsing cost > token savings); reserve model-native encoding for machine-only memory blobs.
5. **Measure with the real tokenizer and golden QA tests per project** — format/compression benchmarks contradict each other; there is no universal optimum, only measured ones.
6. Compression interacts with reasoning cost: report *total* cost (input tokens + induced steps/CoT), not just input shrinkage.