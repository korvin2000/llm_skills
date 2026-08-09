# Synthesis — what 15 research dossiers agree and disagree on

Merged, deduplicated findings from every dossier in `docs/`. Read this instead of the
dossiers. Go to a dossier only for the detail this file points you to.

Source corpus: 15 model-authored dossiers + 1 prior-art survey, ~96K tokens, all dated
2026-08-09. See [INDEX.md](INDEX.md) for the map, [EVIDENCE.md](EVIDENCE.md) for citations.

## Contents

- [1. Notation](#1-notation)
- [2. Settled — build on these](#2-settled--build-on-these)
- [3. Contested — decide before building](#3-contested--decide-before-building)
- [4. Numbers that disagree](#4-numbers-that-disagree)
- [5. Single-source techniques worth stealing](#5-single-source-techniques-worth-stealing)
- [6. Canonical pipeline](#6-canonical-pipeline)
- [7. Open questions](#7-open-questions)

---

## 1. Notation

`[13/15]` = asserted by 13 of 15 dossiers (grep-verified mention count; endorsement noted
where a dossier mentions a technique in order to reject it).
**Verdict:** = the position this project adopts. Change it only with evidence, and record
the change here.

---

## 2. Settled — build on these

Ordered by strength of agreement, not by importance.

### 2.1 Deletion > relocation > rewriting `[15/15]`

Every dossier independently ranks the same three levers in the same order. Sentence-level
rewriting is the *last* and *smallest* lever.

- qwen quantifies it: **~80% deletion, ~15% restructuring, ~5% rewriting**.
- claude: "45–70% reduction, most of it from rules 1–4 rather than from clever wording."
- kimi: "80% of value = delete, not clever rewriting. Most agent files contain 40–60%
  generic/derivable filler."
- ds_flash calls the inversion the single highest-leverage insight in the field.

**Verdict:** A compressor that paraphrases first is doing the bottom 20% of the work.
Order of operations is non-negotiable: **delete → relocate → rewrite → re-encode**.

### 2.2 Progressive disclosure / tiered loading `[13/15]`

Universal four-tier model (names differ, structure identical):

| Tier | What | Cost per session | Budget |
|---|---|---|---|
| T0 always resident | skill `name` + `description`; root rules file | every turn | `description` ≤1024 chars; root file ≤150 lines |
| T1 loaded on trigger | `SKILL.md` body, scoped/nested rules file | once, if relevant | ≤500 lines |
| T2 read just-in-time | `references/*.md`, schemas, runbooks | only if opened | any size; ToC above 100 lines |
| T3 executed, never read | `scripts/*` , validators, linters, CI | **0 tokens** | any size |

**Verdict:** Promoting content from T1 to T3 is the single largest compression move
available. A 400-line validator that prints `OK` beats any prose encoding of the same rules.

### 2.3 Never alter identifiers `[15/15]`

Commands, flags, paths, glob patterns, symbol names, env vars, config keys, ports,
version pins, error strings used for matching, external URLs, numeric thresholds.

**Verdict:** Bit-exact. A pass that changes an identifier is a bug, not a style choice.
Extract an identifier inventory before rewriting; diff it after; fail the run on any delta
in *either* direction (a new identifier is a hallucination).

### 2.4 Delete what the base model already knows `[15/15]`

Explanations of REST/JSON/git/pytest, "write clean code", "follow best practices",
"be thorough". Zero surprisal, non-zero cost — and measurably *negative*, because these
phrases induce extra exploration.

### 2.5 Toolchain-first: never restate an enforced rule `[9/15]`

If a linter, formatter, type checker, hook, or CI gate enforces it, the tool *is* the
constraint. Restating it costs tokens forever, dilutes signal, and creates drift.
claude reports this as the most common real-world smell: **62% of surveyed files**.

Enforcement hierarchy — always use the highest mechanism that works:

```
1. impossible      types, schema, API design, file permissions   0 tokens, 0 failures
2. automatic       formatter, codegen, pre-commit hook           0 tokens
3. checkable       linter, test, CI gate, validator script       ~5 tokens of output
4. runnable        "run scripts/check_x.py"                      ~10 tokens
5. written rule    one line in the rules file                    ~15-40 tokens, forever
6. prose           a paragraph                                   avoid
```

### 2.6 Position matters — attention is U-shaped `[9/15]`

Primacy and recency zones are used reliably; the middle is not. Hard constraints and
most-violated rules go at the top; reference material in the middle; acceptance criteria
at the tail. Every line added to the middle pushes existing lines deeper into the weak zone.

### 2.7 Verification is mandatory `[8/15]`

Compression without verification is vandalism. Consensus test ladder, cheapest first:

1. **Identifier diff** — invariant set before == after. Cheap, mandatory, automatable.
2. **Fact recall** — 10–30 questions generated from the *original*, answered from the
   *compressed* file only. Must-keep facts require 100%.
3. **Rule-enumeration probe** — fresh model lists every rule it would follow; compare to
   intent. Catches silent omission (the dominant failure mode under instruction load).
4. **Idempotency** — second run must be byte-identical.
5. **Behavioural A/B** — 3–5 real tasks with original vs compressed. The only test that can
   show compression *improved* behaviour, which it frequently does.

### 2.8 Structure beats prose for the same content `[12/15]`

Decision tables for branching policy, pseudocode for procedures, type signatures for APIs,
formulas for quantitative rules, grammars for output formats. All are denser *and* less
ambiguous than the prose they replace, and all sit closer to the model's training
distribution than English paragraphs.

### 2.9 Pointer over payload `[13/15]`

`src/auth/session.py::refresh_token` beats a pasted function. `rg -n "CREATE TABLE"
migrations/` beats a schema dump. ~10 tokens, never stale. Prefer symbol references and
grep patterns over line numbers — line numbers rot on the next commit.

### 2.10 Every link needs a pitch `[10/15]`

`path — what's inside — when to read it`. A bare path is either ignored or eagerly slurped;
both are failures. References must be **one level deep** from the entry file — agents
preview nested chains with `head -100` and silently work from partial information.

### 2.11 Markdown is the right default `[11/15]`

Strong training prior, greppable, diffable, cheap structure. HTML is the expensive one.
Exotic serializations (TOON, CSV, custom DSLs) are for bulk uniform *data payloads*, never
for rules an agent must obey. **Consistency within a document matters more than the format
chosen** — mixing separators section-to-section measurably shifts results.

### 2.12 Cache-aware ordering is free quality `[9/15]`

Prompt caching keys on an exact prefix. Stable content (invariants, commands) at the top;
volatile content (sprint focus, WIP notes) at the bottom. Editing line 3 invalidates the
whole file's cache; editing the last line invalidates almost nothing. This is also an
argument for compressor **idempotency** — a non-idempotent pass thrashes the cache on
every run.

---

## 3. Contested — decide before building

These are real disagreements, not phrasing differences. Each needs a project decision.

### 3.1 Symbolic / telegraphic notation — the biggest split

| Position | Dossiers | Claim |
|---|---|---|
| **Pro** | ds_pro, glm52, gemini, hy3, m3, spark1 | Telegraph English ~50% reduction at 99.1% fidelity; predicate logic (`P ⇒ Q`, `∀x`) 69% reduction; symbols are dense and unambiguous |
| **Con** | claude, codex, qwen, fable | Telegraphic English saves ~10% and *increases misparses*; custom grammar is measured **net-negative** (delimiter tokens + the model must learn your grammar before using the content); the compression cliff is real |
| **Conditional** | grok, kimi | Fine for private memory/compaction layers; risky for shared team files unless every consumer is tested |

**Verdict (adopted):** Default **off**. In-distribution symbols only (`->`, `<=`, `|`,
`:=`, regex, CLI flags) — these are cheap and already in the training distribution.
Invented notation, emoji legends, and predicate-logic rewrites are an opt-in
`--aggressive` mode gated behind a measured A/B, and never for shared or safety-critical
files. Rationale: the pro-side numbers are all single-source and unreplicated; the con-side
includes the two most detailed dossiers and a specific mechanism for the harm.

### 3.2 Negative rules: keep them, but how to phrase them?

Two distinct questions get conflated:

- **Are NEVER-rules high value?** Unanimous **yes**. kimi: "negative knowledge is the
  highest-density content — cut positive obvious rules before touching a single gotcha."
  fable: "a compressed doc should preserve NEVER-rules preferentially over SHOULD-rules."
- **Should a rule be *phrased* as a negation?** Split. The Pink Elephant argument
  (`[4/15]` — ds_pro, qwen, mimo, glm52) says naming the forbidden thing activates it.
  fable argues negative instructions are more effective because they target the specific
  mistake the agent keeps making.

**Verdict (adopted):** Prefer positive replacement where a real alternative exists
(`Use httpx` beats `Don't use requests`). Reserve explicit `NEVER` for genuine cliffs —
data loss, security, irreversibility — and cap that list at ~7 items so it stays salient.
Never delete a gotcha to satisfy a phrasing preference.

### 3.3 Deliberate top-and-bottom repetition of critical rules

- **Pro** `[5]`: claude (T19, ~30 tokens, worth it), kimi, m3, glm52, hy3 — primacy covers
  the top, recency the bottom, the middle is where rules die.
- **Con** `[1, explicit]`: codex — "do not duplicate every critical instruction at both top
  and bottom; duplication costs tokens and can drift."

**Verdict (adopted):** Repeat the top **three** safety-critical rules only, as a ≤5-line
closing block, generated from the top block so the two cannot drift.

### 3.4 Non-English documentation for token density

hy3 alone proposes a CJK trick (`禁:删/产/密` ≈ 5 tokens vs ~12 English). Rejected by
claude, codex, fable, qwen, grok — qwen calls it a **myth** outright ("Chinese saves
nothing; tokenizer fragmentation eats ideograph density").

**Verdict (adopted):** Instruction core is English. Identifiers are English anyway; a
two-language document breaks grep and makes diffs unreviewable. Exception: few-shot
examples for output the model must *produce* in language X.

### 3.5 Emoji as semantic tags

glm52 and hy3 advocate emoji as 1–2 token semantic anchors with high attention weight.
claude and qwen measure them at 2–4+ tokens each with zero signal, and argue an emoji
legend is a worse table.

**Verdict (adopted):** No. If you are writing "Legend: 🔴 = …", you have invented a worse
table. ASCII keywords (`MUST`, `NEVER`, `ASK`) do the same job, grep cleanly, and diff.

### 3.6 Diagrams: Mermaid vs edge lists vs images

Near-unanimous that *images* are wrong for machine-read docs (not fetched, not greppable,
not diffable, expensive when they are fetched). Split on Mermaid: ds_flash and spark1 like
it ("renders for humans, stays text for agents"); claude, codex, qwen prefer a 4-line edge
list (`ingest -> validate -> normalize -> persist`), which is ~10× cheaper.

**Verdict (adopted):** Edge list / arrow chain in the agent-facing file; Mermaid in the
human README or a T2 `.mmd` file. Exception: genuinely visual tasks (UI layout, PDF form
geometry) where letting the model *look* is the point.

---

## 4. Numbers that disagree

Direction is consistent; magnitude is not. **Do not hard-code any of these.** Measure with
the target tokenizer on the target corpus.

| Quantity | Reported range | Spread | Note |
|---|---|---|---|
| Root rules file budget | 20–30 lines (mimo, small repos) → 200 lines (nvidia) | **10×** | Converges on **≤150 lines** for a normal repo |
| Root file token budget | 200–600 tok (m3) → 2000 tok (hy3) | 10× | gemini: <800, hard ceiling 1200 |
| Always-on imperative budget | ≤40 (claude) → 150–200 (ds_pro) | 5× | Different things measured: claude's is a target, ds_pro's is where adherence *starts* collapsing. gemini splits the difference: 150–200 total slots minus ~50 for scaffolding = **80–120** available |
| HTML → Markdown saving | 30% (qwen, tables only) → 90% (fable, ds_flash) | **3×** | Direction unanimous, magnitude unusable as a constant |
| Telegraphic rewrite saving | ~10% and harmful (claude) → 30–50% (gemini) | 5× | qwen's −15–21% is the best-attested midpoint, explicitly framed as "not −75%" |
| Overall compression target | 20–40% safe → 60–95% aggressive | 4× | Most defensible: **40–65% balanced**, >5× on instructions usually net-negative |
| Prose → table saving | 40% (nvidia) → 63% (gemini) | 1.6× | Consistent enough to trust the direction |

### 4.1 The one genuine contradiction in the evidence

Two headline studies point opposite ways on cost:

- **arXiv 2602.11988 (ETH):** context files increase inference cost **+20–23%** and
  LLM-generated ones reduce success in 5 of 8 settings.
- **arXiv 2601.20404:** presence of `AGENTS.md` gives **−28.6% median runtime** and
  fewer output tokens (kimi reports −20%, mimo reports −16.6% — themselves inconsistent).

kimi supplies the reconciliation and it is the most important sentence in the corpus:

> **The failure mode is bloat, not the format.** A well-formed compact file is measurably
> better than none; a bloated or auto-generated one is measurably worse than none.

**Verdict:** This is the project's thesis. The skill's job is to move files from the second
category to the first, and the primary metric is therefore *cost per completed task*, not
tokens in the source file.

---

## 5. Single-source techniques worth stealing

High value, only one dossier proposes each. Unreplicated — treat as hypotheses to test.

| Technique | Source | Why it matters |
|---|---|---|
| **Amnesia probe** | claude §8.6 | Per-claim empirical redundancy detection: ask a fresh cheap model the question a claim answers, with no doc. Answer matches → AMBIENT, delete. Answer contradicts → HIGH VALUE, promote. Model asks → keep. Uses the model as the compressor's dictionary; fully automatable |
| **Expected token residency** | codex §20 | `P(load) × tokens` per unit. A 300-token overview loaded every task costs 4× a 1,500-token reference loaded 5% of the time. The cleanest single metric for restructuring decisions |
| **Instruction ablation** | codex §19 | Treat each rule as a feature: `Δ(r) = Eval(with r) − Eval(without r)`. `Δ≈0` → remove; `Δ<0` → the rule is *harmful*. Operationalizes the ETH finding directly |
| **Removal ledger** | claude §8.10 | `AGENTS.notes.md`, not loaded by any agent, recording what was removed and why. Zero context cost. Without it the file re-inflates within two quarters |
| **Docs-as-tests** | claude §8.9 | Extract every command and path from the file; CI runs `--help` and `test -e`. A documentation line CI can verify never rots. Converts doc maintenance from discipline into a build failure |
| **Semantic checksum** | codex §21 | Extract `must / never / numbers / paths / precedence / fallbacks` to YAML before and after; fail on missing literal, changed modality, changed number, lost exception, widened scope |
| **Co-access graph partitioning** | codex §18 | Partition by `P(a and b needed in same task)` mined from traces, not by topic. Two topics may always be needed together; two subsections of one topic may have very different activation rates |
| **Sort by violation frequency** | claude §8.12 | Mine transcripts for which rules were actually broken; order by `violations × cost`. The only honest way to decide what earns a MUST |
| **Hybrid storage** | glm52 §12.1 | Compressed in context, original on disk, explicit fallback pointer. Makes compression *reversible* — the safest form |
| **HTML comments as free human notes** | kimi §7.1 | Claude Code strips block-level `<!-- -->` from CLAUDE.md before injection. Rationale for maintainers at zero model cost. **Verify before relying on it** |
| **Compressor must not invent** | kimi §7.11 | The skill may delete, merge, reorder, relocate, re-encode. It may **not add facts**. Gaps get flagged as `<!-- GAP: no test command -->`, never filled with generic prose |
| **Rejected alternatives are the highest-value memory** | qwen §8 | "Evaluated X, rejected because Y" — git records decisions but cannot record non-decisions |
| **Unhobbling pass** | fable §6k | Defensive rules written for weaker models become negative-value as models improve. Flag `always confirm before X` for review against current capability |
| **State → query instruction** | codex §23 | Replace "Current packages are A 1.2, B 4.5" with "Read versions from `package.json`; never infer them from this file." Shorter, no staleness, single source of truth |
| **Two-stage skill reduction** | available_skills (SkillReducer) | Stage 1 optimizes the routing `description`; stage 2 classifies the body into actionable-core / supplementary / removable. Reported 48% description + 39% body compression with **+2.8% functional quality** |

---

## 6. Canonical pipeline

Merged from 15 independently proposed pipelines. They differ in naming, not in shape.

```
STAGE 0  BASELINE
  count tokens / lines / imperatives / sections / links with the real tokenizer
  extract INVARIANT SET (commands, paths, symbols, versions, error strings, URLs)
  snapshot original (git, or .orig copy)
  if agent transcripts available: mine violated rules + failure modes

STAGE 1  DETECT                                        # cheap, mechanical, explainable
  lint leakage    rules already enforced by ruff/eslint/prettier/tsconfig/editorconfig
  redundancy      claims already in README, docs/, code comments, another rules file
  skill leakage   sections gated on a rare path / used by <20% of tasks
  blind refs      links with no "what + when" pitch
  conflicts       (scope, subject) triples with incompatible directives
  fossilization   git log: single commit, or last-modified << code churn
  bloat           >200 lines, >12 sections, >2 nesting levels
  ambient         amnesia probe with a cheap model
  -> REPORT, and get approval for anything CONFLICTING or ambiguous

STAGE 2  RESTRUCTURE                                   # move before you shrink
  rarely-used sections   -> T2 reference file + pitched link
  deterministic rules    -> T3 script / linter config / CI gate
  payloads               -> pointers (path::symbol, rg pattern, command)
  scope-local rules      -> nested rules file in the subtree that owns them
  order: stable invariants -> hot rules -> reference -> volatile

STAGE 3  REWRITE                                       # per-class budgets, not uniform
  commands/paths/invariants   0%      lossless-only zone
  prohibitions/safety         0-10%   wording carries the force
  procedures                  20-40%  keep steps, cut narration
  reference tables            20-40%  or relocate whole to T2
  examples                    40-70%  keep 2-3 diverse, drop near-duplicates
  rationale                   60-90%  keep only rationale that generalizes
  overview/motivation         90-100% usually deletable outright
  loop: rewrite ~20% shorter, re-extract directives; revert and stop when one is lost

STAGE 4  ENCODE                                        # smallest lever, do it last
  ASCII punctuation, flatten nesting >2, drop decoration, table-ify repeats

STAGE 5  REORDER
  most-violated rule first; <=5-line non-negotiables recap last; stable prefix for cache

STAGE 6  VERIFY
  identifier diff -> fact recall -> rule enumeration -> idempotency
  optional: behavioural A/B on 3-5 tasks

STAGE 7  EMIT
  compressed files
  removal ledger (not agent-loaded)
  report: before/after tokens, lines, imperatives; per-item rationale; open questions
```

**Hard constraints on any implementation:** it may delete, merge, reorder, relocate and
re-encode. It may **not add facts**. It may **not auto-resolve a contradiction** —
automated conflict detection runs at ~57% precision in published work, so conflicts go to
a human. Any new claim is flagged, never written.

---

## 7. Open questions

Unresolved by the corpus. Each blocks a design decision.

1. **Which harnesses lazy-load references?** codex §17 warns that Markdown *imports* are
   often eager while *links* are lazy. Splitting a file is not compression if every piece
   loads at startup. Must be verified per harness before the restructure stage can be
   trusted.
2. **Is the `<!-- -->` stripping behaviour real and current?** kimi's zero-cost-comment
   trick depends on it. Unverified, single-source, and would change how rationale is stored.
3. **What is the actual instruction budget?** ≤40 vs 80–120 vs 150–200 differ by 5×. This
   sets the primary red-line in the scoring rubric and cannot stay a guess.
4. **Do the 2026 arXiv citations resolve?** 41 unique IDs, 26 of them cited by exactly one
   LLM-authored dossier. See [EVIDENCE.md](EVIDENCE.md) — none are verified yet.
5. **Does symbolic notation help or hurt on current models?** §3.1 is decided on the
   balance of argument, not on measurement. One A/B would settle it.
6. **What is the target harness set?** Claude Code only, or Codex + Cursor + Copilot too?
   Determines whether the skill emits `CLAUDE.md`, `AGENTS.md`, or both, and whether nested
   scoping is available.
