# Corpus index

Map of `docs/`. Every entry states **what is inside** and **when to open it**, so you can
route without reading. Sizes are estimates at ~3.9 chars/token.

**Read [SYNTHESIS.md](SYNTHESIS.md) first.** It merges all 15 dossiers. Open an individual
dossier only when the synthesis points you at one, or when you need a worked example the
synthesis compressed away.

## Contents

- [Derived documents](#derived-documents)
- [Routing table — which dossier answers which question](#routing-table--which-dossier-answers-which-question)
- [Dossiers by signal](#dossiers-by-signal)
- [Corpus facts](#corpus-facts)
- [Search recipes](#search-recipes)

---

## Derived documents

Written by this project. These supersede the raw dossiers for day-to-day work.

| File | Lines | What's inside | When to read |
|---|---:|---|---|
| [SYNTHESIS.md](SYNTHESIS.md) | 366 | Consensus findings, the 6 real disagreements with adopted verdicts, contradictory numbers, 15 single-source techniques, the canonical 8-stage pipeline, open questions | **Always, first.** Before any design decision |
| [EVIDENCE.md](EVIDENCE.md) | 152 | Citation ledger: 41 arXiv IDs with citing dossiers and verification status, known citation conflicts, primary vendor docs, prior-art repos | Before citing a number externally, or when a claim looks too good |
| [INDEX.md](INDEX.md) | this | Corpus map and routing | When the synthesis points you to a dossier and you need to pick one |

---

## Routing table — which dossier answers which question

| Question | Go to |
|---|---|
| What does the empirical evidence actually say? | `claude-opus.md` §1, `kimi.md` §1, `qwen.md` §1 |
| How do I detect a bad context file mechanically? | `claude-opus.md` §1.2 (6 smells) + Appendix A (ripgrep recipes) |
| What is the formal objective being optimized? | `codex.md` §1, `hy3.md` §0 (rate–distortion), `mimo.md` §1 |
| How do I decide hot / warm / cold placement? | `codex.md` §3 + §20 (expected token residency) |
| What are the concrete rewrite transformations? | `claude-opus.md` §6 (T1–T20), `codex.md` §10 (R1–R10), `nvidia.md` §2 |
| Deterministic, regex-safe cleanup rules? | `ds_pro.md` §II, `glm52.md` §4.1, `nvidia.md` §2 (tier 1) |
| How do I verify I didn't break anything? | `claude-opus.md` §9, `codex.md` §21+§32, `nvidia.md` §9 |
| What are the budgets and hard limits? | `claude-opus.md` §10, `kimi.md` §5.2, `qwen.md` §1 |
| Symbolic / Telegraph-English notation — the pro case | `ds_pro.md` §V.1, `glm52.md` §5.3, `gemini.md` §3.3 |
| Symbolic notation — the con case | `claude-opus.md` §7–8.2, `codex.md` §27, `qwen.md` §5 |
| Memory-file specific compaction | `codex.md` §6+§25, `qwen.md` §8, `gemini.md` §4, `grok.md` §5H–I |
| `SKILL.md` authoring and progressive disclosure | `ds_flash.md` §5, `qwen.md` §9, `fable.md` §5 |
| Format benchmarks (MD / YAML / JSON / TOON / CSV) | `kimi.md` §4.1, `qwen.md` §6, `glm52.md` §2.2 |
| Tokenizer mechanics and per-format token costs | `qwen.md` §2, `hy3.md` §1, `ds_flash.md` §3 |
| Prompt-cache interaction | `claude-opus.md` §4.6, `hy3.md` §4.2, `m3.md` §4.2, `kimi.md` §7.10 |
| What already exists that I could fork? | `available_skills.md` (whole file) |
| Skill directory layout proposals | `claude-opus.md` §12.1, `codex.md` §35, `spark1.md` §9 |
| Ready-to-adapt `SKILL.md` drafts | `claude-opus.md` §12.2, `codex.md` §36, `kimi.md` §8, `qwen.md` §11 |
| Worked before/after examples | `claude-opus.md` §13, `gemini.md` §7, `nvidia.md` §2, `glm52.md` §8 |
| Raw source links for further research | `reasearch_links.md` |

---

## Dossiers by signal

Signal rating reflects density of non-duplicated, actionable, evidence-linked content —
judged after reading all 17 files. It is not a judgement of the authoring model.

### Tier A — read in full if you read anything

| File | Tokens | Distinctive contribution |
|---|---:|---|
| `claude-opus.md` | ~14.6K | The most complete field manual. Only source for: the 6 configuration smells as a detector set, the amnesia probe, the removal ledger, docs-as-tests, generational-loss/idempotency, per-section compression budgets, and ~30 ripgrep detection recipes in Appendix A. Opinionated with explicit **Verdict:** markers |
| `codex.md` | ~12.7K | The most rigorous framing. Only source for: expected token residency, instruction ablation, co-access-graph partitioning, semantic checksums, eager-import vs lazy-link warning, and a 5-level implementation roadmap (v0→v4) |
| `kimi.md` | ~5.5K | Best evidence-per-token. Uniquely reconciles the two contradictory headline studies, best format-selection matrix, tokenizer-aware writing rules, the "compressor must not invent" invariant |
| `qwen.md` | ~4.7K | Best calibrated numbers. Quantifies the 80/15/5 split, debunks the CJK-density myth, warns that custom grammar is measured net-negative, strongest memory-file section |

### Tier B — targeted reading

| File | Tokens | Distinctive contribution |
|---|---:|---|
| `glm52.md` | ~8.5K | The SDE (semantic-density) formula, position-aware truncation, the protected-facts block, md2idx index-then-retrieve pattern |
| `ds_pro.md` | ~5.9K | Best deterministic lossless rule catalog (14 named rules with savings estimates), LLMD line-prefix format, dictionary dedup with the LZ77 break-even formula |
| `nvidia.md` | ~5.7K | Best two-tier lossless/lossy table with per-technique reduction figures, embedded compression directives, evaluation-metric table, config schema |
| `fable.md` | ~5.7K | Strongest on migrating rules to hooks/CI (25–40% vs ~95% compliance), the "unhobbling" pass, calibrate-to-weakest-model rule |
| `ds_flash.md` | ~3.5K | Tightest summary of the whole field. Representation-selection decision table, abbreviation-hazard warning, HTML→MD token accounting |
| `available_skills.md` | ~3.3K | Prior-art survey with assessments: caveman-compress, Agent-Skills-for-Context-Engineering, SkillReducer, optimize-agent-docs. Read before writing any code |

### Tier C — spot checks and second opinions

| File | Tokens | Distinctive contribution |
|---|---:|---|
| `gemini.md` | ~5.6K | Instruction-slot capacity model (80–120 usable), `Rule → Action → Verify` pattern, clean before/after case studies |
| `mimo.md` | ~4.7K | Rate–distortion theory framing, three-compression-families comparison, context-budget manager with pinned facts |
| `m3.md` | ~4.4K | Referential compression, delta-from-default, hashed/versioned references, KV-cache-friendly formatting |
| `hy3.md` | ~3.8K | Tokenizer/BPE mechanics, tiered memory (MemGPT lineage), typed-constraint specs. **Contains the rejected CJK-density proposal** (see SYNTHESIS §3.4) |
| `spark1.md` | ~3.6K | `llm-min.txt` 6-stage condensation pipeline, llms.txt / llms-full.txt pairing, layered-markdown dual-audience trick |
| `grok.md` | ~3.5K | Compact and well-structured; good AGENTS.md skeleton and reviewer checklist. Mostly overlaps Tier A |
| `reasearch_links.md` | ~0.5K | 26 raw URLs, unannotated. Superseded by EVIDENCE.md except as a to-read queue |

---

## Corpus facts

- **17 files, 8,107 lines, ~96K tokens, ~375 KB.** Reading the corpus costs roughly half a
  200K context window.
- **All dated 2026-08-09**, produced independently by different models answering the same
  research brief. Overlap is therefore *convergent evidence*, not citation chains — that is
  what makes the agreement counts in SYNTHESIS.md meaningful.
- **Redundancy is high by design.** Roughly 60–70% of any Tier-B/C dossier restates Tier-A
  content. The synthesis exists so you never pay that cost twice.
- **These are LLM-authored research notes, not peer-reviewed sources.** Numeric claims and
  citations are unverified. See [EVIDENCE.md](EVIDENCE.md).

### Naming convention

One file per authoring model: `claude`, `codex`, `gemini`, `grok`, `kimi`, `qwen`,
`fable`, `nvidia`, `spark1`, `mimo`, plus abbreviations `ds_pro`/`ds_flash` (DeepSeek),
`glm52` (GLM-5.2), `hy3` (Hunyuan 3), `m3` (Minimax M3). Two files are not dossiers:
`available_skills.md` (prior art) and `reasearch_links.md` (raw links).

---

## Search recipes

Cheaper than opening files.

```bash
rg -l -i "progressive disclosure" docs/          # which dossiers cover a concept
rg -n -i "verdict:" docs/claude-opus.md               # opinionated positions only
rg -n -o 'arxiv\.org/\S+' docs/ | sort -u        # every citation
rg -n -i "never|must not" docs/SYNTHESIS.md      # the hard constraints
rg -n '^#{2,3} ' docs/codex.md                   # section list without reading the file
```
