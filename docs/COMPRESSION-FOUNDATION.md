# Compression Engineering for LLM-Facing Markdown

Independent research foundation for the future `compress-llm-documentation` skill.

**Status:** research and design guidance, not an implemented skill.  
**Independent phase:** completed from all 15 primary dossiers plus the two supporting
source lists before consulting `INDEX.md`, `EVIDENCE.md`, or `SYNTHESIS.md`.  
**Evidence cutoff:** 2026-08-09. Platform behavior can change; re-check official
documentation before encoding it in a skill.

## Contents

1. [Executive verdict](#1-executive-verdict)
2. [Method, evidence, and scoring](#2-method-evidence-and-scoring)
3. [The optimization target](#3-the-optimization-target)
4. [The preservation contract](#4-the-preservation-contract)
5. [Ranked rule catalog](#5-ranked-rule-catalog)
6. [Safe transformation patterns](#6-safe-transformation-patterns)
7. [Artifact and harness profiles](#7-artifact-and-harness-profiles)
8. [Canonical compression pipeline](#8-canonical-compression-pipeline)
9. [Validation and evaluation](#9-validation-and-evaluation)
10. [Smells and detectors](#10-smells-and-detectors)
11. [Evidence assessment](#11-evidence-assessment)
12. [Dossier-by-dossier critical audit](#12-dossier-by-dossier-critical-audit)
13. [Worst practices: reject by default](#13-worst-practices-reject-by-default)
14. [Implications for the future skill](#14-implications-for-the-future-skill)
15. [Open decisions](#15-open-decisions)
16. [Reconciliation with the prior derived documents](#16-reconciliation-with-the-prior-derived-documents)
17. [Primary references](#17-primary-references)

---

## 1. Executive verdict

The best compression is not the shortest text. It is the smallest **task-appropriate,
behavior-preserving context** that the target agent will actually receive, understand,
and use.

That leads to six conclusions:

1. **Protect behavior before reducing tokens.** A constraint, exception, precedence rule,
   literal, or boundary condition can be worth more than a page of background. Compression
   that loses one such atom has failed even if semantic similarity remains high.
2. **Change context architecture before wording.** Removing proven duplicates, separating
   always-needed from conditional material, and making retrieval reliable usually saves
   more expected context than abbreviating prose.
3. **Compile; do not summarize.** Inventory semantic atoms, classify them, transform them,
   and verify the result. A one-shot summary is not a safe compressor.
4. **Use familiar Markdown by default.** Headings, short directives, ordinary lists, and
   selective tables are broadly portable. Symbols, private dictionaries, custom DSLs, and
   token tricks impose decoding and maintenance costs and often reduce reliability.
5. **Treat “lossless” narrowly.** Only transformations proven irrelevant to the target
   renderer and harness are surface-lossless. Comments, frontmatter, examples, ToCs,
   rationale, code comments, and metadata are not intrinsically disposable.
6. **Optimize measured task utility, not a compression ratio.** The production measure is
   cost per successful task, including retrieval, retries, re-reading, and repair.

The priority order is:

> correctness and safety > instruction availability > discoverability > maintainability
> > expected context cost > raw file size > visual terseness

The corpus strongly converges on deletion, progressive disclosure, explicit constraints,
and evaluation. That convergence is useful design evidence, but not 15 independent
experiments: every dossier answered the same brief, many repeat the same citations, and
several repeat the same unsupported numerical claims.

## 2. Method, evidence, and scoring

### 2.1 Independent-first procedure

The analysis used this order:

1. Freeze the evaluation rubric.
2. Read every primary dossier in full:
   `claude-opus.md`, `codex.md`, `ds_flash.md`, `ds_pro.md`, `fable.md`,
   `gemini.md`, `glm52.md`, `grok.md`, `hy3.md`, `kimi.md`, `m3.md`,
   `mimo.md`, `nvidia.md`, `qwen.md`, and `spark1.md`.
3. Read `available_skills.md` as a prior-art survey and `reasearch_links.md` as an
   unvetted lead queue.
4. Normalize repeated wording into distinct technique families. “Use tables,” “replace
   prose with matrices,” and “table-over-paragraph” are one family, not three votes.
5. Check load-bearing research claims against primary papers or official platform
   documentation.
6. Produce the independent conclusions in ``1-15.
7. Only then inspect `INDEX.md`, `EVIDENCE.md`, and `SYNTHESIS.md` and record the
   comparison in `16.

This prevents agreement with an earlier synthesis from being mistaken for independent
convergence.

### 2.2 Evidence grades

| Grade | Meaning | Appropriate use |
|---|---|---|
| **A** | Direct, relevant controlled evidence or current authoritative platform behavior | Default rule when the result transfers to the target |
| **B** | Relevant observational study, reproducible tool evaluation, or strong official practice guidance | Strong default with stated scope |
| **C** | Adjacent experiment, narrow preprint, practitioner evidence, or a plausible mechanism | Conditional rule; validate locally |
| **D** | Corpus consensus without direct evidence, unverified statistic, anecdote, or speculative technique | Hypothesis only |
| **X** | Contradicted, semantically unsafe, imaginary, or based on an invalid inference | Reject by default |

“Current official behavior” earns high confidence only for what the harness does now. It
does not prove that the behavior is optimal, portable, or stable.

### 2.3 Effectiveness and risk scales

**Effectiveness (E)** estimates improvement in expected context cost or agent performance:

- **E4:** architectural or fidelity-critical; can change most task runs.
- **E3:** high-value default.
- **E2:** useful for a matching content shape.
- **E1:** small or mostly cosmetic.
- **E0:** no reliable benefit; may be negative.

**Information-loss risk (R)** assumes the technique is applied by a competent but fallible
compressor:

- **R0:** presentation-only and mechanically checkable.
- **R1:** low after literal and structural validation.
- **R2:** conditional; semantic review required.
- **R3:** high; behavioral evaluation and explicit approval required.
- **R4:** unacceptable as a default.

**Portability (P)** is high, medium, or low across models and harnesses. A technique can be
effective on one platform and still have low portability.

### 2.4 Decision labels

- **P0 — prerequisite:** protects correctness; run before compression.
- **P1 — default:** high value and acceptably safe with normal validation.
- **P2 — conditional:** apply only when the content shape and harness fit.
- **P3 — experimental:** isolate, evaluate, and keep reversible.
- **Reject:** never apply automatically.

The labels are not averages. A high-risk technique does not become safe because many
dossiers repeat it.

## 3. The optimization target

### 3.1 Objective

For sections `i` with load probability `p_i`, token cost `t_i`, retrieval cost `r_i`,
and expected rework from omission `w_i`:

`ExpectedCost = sum_i p_i * (t_i + r_i + w_i)`

Minimize that cost **subject to**:

- preserved hard constraints and permissions;
- preserved decisions, conditions, exceptions, and precedence;
- preserved task success and safety;
- sufficient discoverability for conditional material;
- acceptable human maintenance and auditability.

This model comes from the corpus’s strongest conceptual work on expected residency,
discoverability tax, and semantic checksums (`codex.md `20-22`), reinforced by
tokens-per-task reasoning in the context-compression prior art
(`available_skills.md `2-3`). It also corrects the common error of optimizing only
bytes, one request, or one model response.

### 3.2 Five distinct operations

| Operation | What changes | Typical risk |
|---|---|---|
| **Surface normalization** | Whitespace or syntax with proven-equivalent rendering | R0-R1 |
| **Lexical compression** | Wording, sentence shape, local repetition | R1-R2 |
| **Semantic compression** | Which facts, rules, rationale, and examples remain | R2-R4 |
| **Architectural compression** | What is always loaded, scoped, linked, generated, or executed | R1-R3 |
| **Operationalization** | Prose becomes a script, config, schema, hook, or test | R1-R3 |

Calling the last four “lossless” hides the actual decisions. A transformation is safe
only relative to a declared preservation contract and target harness.

### 3.3 What is out of scope by default

This foundation concerns durable, LLM-facing Markdown: `AGENTS.md`, `CLAUDE.md`,
`SKILL.md`, memory, rules, and their references.

Adjacent techniques need separate modes:

- compressing a transient RAG prompt with LLMLingua;
- compacting a live conversation history;
- converting presentation-heavy HTML into Markdown;
- minifying source code or tool schemas;
- shortening the agent’s user-facing answers.

They may share mechanisms, but they have different invariants and evidence.

## 4. The preservation contract

### 4.1 Semantic ledger

Before rewriting, extract every operational unit into a ledger. One row should record:

| Field | Preserve when present |
|---|---|
| **ID and source** | Stable local ID plus `file.md `section` provenance |
| **Type** | instruction, fact, decision, definition, example, rationale, warning, pointer |
| **Actor and scope** | who acts; repository, path, task, file type, platform, or lifecycle scope |
| **Trigger** | the condition or event that activates the unit |
| **Modality** | MUST, MUST NOT, SHOULD, MAY, default, preference, or observation |
| **Action and object** | exact required or prohibited behavior |
| **Exceptions** | cases where the main rule does not apply |
| **Precedence** | which rule wins when scopes overlap |
| **Default and fallback** | what happens when no branch matches or a step fails |
| **Order and dependency** | required sequencing, prerequisites, and concurrency limits |
| **Verification and stop** | completion signal, check, escalation, or abort condition |
| **Exact anchors** | commands, flags, paths, globs, identifiers, keys, versions, numbers, units, URLs, error text, schemas |
| **Rationale** | why the rule exists when that knowledge prevents unsafe generalization |
| **Status** | current, superseded, disputed, uncertain, or externally enforced |

This is a concrete synthesis of invariant sets and protected facts
(`claude-opus.md `2.4`; `glm52.md `9`), the semantic IR
(`codex.md `8`), and fact inventories (`kimi.md `6`).

### 4.2 Protected atoms

Never silently delete or paraphrase:

- MUST / NEVER / ONLY / ASK / DO NOT semantics;
- scope, conditions, exceptions, and precedence;
- authorization and destructive-action boundaries;
- commands, flags, paths, globs, identifiers, environment variables, config keys,
  output schemas, error strings, URLs, numbers, units, and version constraints;
- required order, retries, fallbacks, verification, and stop conditions;
- examples whose boundary case or exact output is the specification;
- non-obvious gotchas that the agent cannot know to retrieve;
- unresolved disagreements and known gaps.

Preserve exact wording only where wording is itself contractual. Preserve exact meaning
and literals elsewhere.

### 4.3 Examples are sometimes executable specification

An example is redundant only if all of its behavior is stated elsewhere and validated.
It is semantic when it reveals:

- an edge or failure case;
- exact syntax or output shape;
- ordering or precedence;
- a non-obvious combination of rules;
- an implicit product requirement.

This corrects the corpus’s recurring “examples are removable payload” claim. SkillReducer
found that relocating examples was a recurring source of regressions; diverse canonical
examples remain valuable (`claude-opus.md `6 T15`; `fable.md `4`; `kimi.md `3.2`).

### 4.4 Rationale is conditional, not decorative

Keep the shortest rationale that changes future decisions. Remove motivation that merely
restates the rule.

Keep rationale when it:

- distinguishes an invariant from a style preference;
- explains a security, data-loss, compatibility, or performance cliff;
- tells the agent when a rule may safely be generalized;
- records why an apparently simpler alternative was rejected.

The safe form is usually `Rule — because consequence`, not a historical essay
(`claude-opus.md `5.3`; `glm52.md `12.5`).

### 4.5 Conflicts and gaps

Do not let the compressor choose silently between incompatible rules. Record:

1. both source statements;
2. their scopes and authority;
3. whether one is demonstrably stale;
4. the proposed verdict and reason;
5. the required human decision if authority is unclear.

If a needed fact is absent, write `<!-- GAP: ... -->`. Never invent a command,
condition, threshold, or process to make the compressed document look complete.

## 5. Ranked rule catalog

The catalog deduplicates the corpus into distinct recommendation families. Ratings are
defaults; the target artifact and harness can change them.

### 5.1 P0 — fidelity prerequisites

| ID | Rule | E | R | Ev | P | Verdict |
|---|---|---:|---:|---|---|---|
| P0.1 | Identify artifact type, intended readers, target models, harness, load semantics, and authority before editing | 4 | 0 | A | H | Mandatory; architecture is otherwise guesswork |
| P0.2 | Inventory every source and determine canonical vs generated vs historical status | 4 | 0 | B | H | Mandatory source graph |
| P0.3 | Build the semantic ledger in `4.1 before deleting or paraphrasing | 4 | 1 | B | H | Core correctness mechanism |
| P0.4 | Extract exact anchors separately and compare them after transformation | 4 | 0 | B | H | Mechanically catches many silent losses |
| P0.5 | Detect conflicts, overlapping scopes, stale copies, and precedence gaps | 4 | 0 | B | H | Never auto-resolve uncertain authority |
| P0.6 | Separate facts, instructions, examples, rationale, pointers, and enforcement | 3 | 0 | B | H | Prevents format-blind deletion |
| P0.7 | Record baseline bytes, lines, actual target-tokenizer counts when available, and behavior | 3 | 0 | B | H | Ratios without a baseline are meaningless |
| P0.8 | Define validation tasks and failure thresholds before compression | 4 | 0 | A | H | Prevents metric shopping |
| P0.9 | Keep the change diffable and reversible through version control or an explicit artifact | 3 | 0 | B | H | Prefer repository history over runtime sidecar clutter |
| P0.10 | Mark missing information; never synthesize new operational facts | 4 | 0 | A | H | Any invention is a failed compression |

### 5.2 P1 — high-value defaults

| ID | Rule | E | R | Ev | P | Verdict |
|---|---|---:|---:|---|---|---|
| P1.1 | Delete exact duplicate payload after selecting one authoritative home and preserving a useful pointer where needed | 4 | 1 | B | H | Highest-confidence semantic saving |
| P1.2 | Delete generic AI advice, throat-clearing, self-evident praise, repeated introductions, and empty conclusions | 3 | 1 | B | H | Keep any embedded exception or scope |
| P1.3 | Remove content proven stale or superseded; retain provenance when future readers need it | 4 | 1 | B | H | “Old” is evidence-based, not age-based |
| P1.4 | Replace repeated terminology with one canonical term; define unavoidable aliases once | 3 | 1 | B | H | Do not abbreviate contractual identifiers |
| P1.5 | Rewrite local prose as explicit `condition -> action` directives | 3 | 1 | B | H | Preserve actor, modality, and exceptions |
| P1.6 | Express a normal case once, followed by explicit exceptions and fallback | 3 | 1 | B | H | Safer than repeating full rules |
| P1.7 | Hoist a condition shared by several adjacent rules | 2 | 1 | B | H | Do not widen its scope |
| P1.8 | Close enumerations when the list is exhaustive; state stop and escalation conditions | 3 | 1 | B | H | Reduces improvisation |
| P1.9 | Use descriptive headings as retrieval keys and keep hierarchy shallow but truthful | 3 | 0 | B | H | Headings carry semantic structure |
| P1.10 | Give every pointer a `what + when` pitch | 4 | 1 | B | H | A bare path creates discoverability tax |
| P1.11 | Keep always-relevant rules hot; move genuinely conditional procedures and references to verified on-demand scope | 4 | 2 | A | M | High leverage; harness-dependent |
| P1.12 | Keep non-obvious gotchas in the smallest context guaranteed to load before the mistake | 4 | 1 | B | H | Do not rely on retrieval when trigger is unknowable |
| P1.13 | Move repeatable multi-step methods into a skill when the target supports skill activation | 4 | 2 | A | M | Root retains trigger and routing |
| P1.14 | Move deterministic enforcement into config, schemas, hooks, linters, tests, or scripts when those mechanisms truly run | 4 | 2 | B | M | Keep invocation, scope, and important rationale |
| P1.15 | Compress memory into current facts, decisions, open items, artifacts, and superseded state rather than chronology | 4 | 2 | B | M | Preserve provenance and unresolved constraints |
| P1.16 | Chunk on behavioral or task boundaries, not arbitrary token counts | 3 | 1 | B | H | Each retrieval unit needs sufficient local context |
| P1.17 | Keep the smallest diverse set of canonical success, boundary, and failure examples | 3 | 2 | A | H | Validate before removing the rest |
| P1.18 | Normalize safe whitespace: trailing spaces without line-break meaning, excessive blank lines, and inconsistent heading spacing | 1 | 0 | B | H | Protect hard breaks, fences, YAML, and render-sensitive text |
| P1.19 | Remove navigation or decoration only after proving it carries no routing, status, ownership, or semantic metadata | 2 | 1 | C | M | Role-aware, never blanket |
| P1.20 | Measure expected loaded tokens and retrieval/rework, not only file bytes | 4 | 0 | B | H | Central architectural metric |

### 5.3 P2 — conditional transformations

| ID | Technique | E | R | Ev | P | Use only when |
|---|---|---:|---:|---|---|---|
| P2.1 | Bullets instead of paragraphs | 2 | 1 | B | H | Units are parallel and relationships remain explicit |
| P2.2 | Mapping or comparison table | 2 | 2 | C | M | Rows share a stable schema; prose would repeat fields |
| P2.3 | Decision table | 3 | 2 | C | M | Conditions and outcomes are finite and non-overlapping |
| P2.4 | Guarded pseudocode | 3 | 2 | C | M | Ordering and branches matter, and all failure paths fit |
| P2.5 | Grammar or EBNF | 3 | 2 | C | M | Exact syntax is the subject; accompany unfamiliar semantics |
| P2.6 | Type signature or schema | 3 | 2 | B | M | Data shape is exact and the notation is native to the audience |
| P2.7 | Formula | 2 | 2 | C | M | The relationship is truly quantitative, not prose disguised as math |
| P2.8 | ASCII or Mermaid diagram | 2 | 2 | C | M | Topology or flow is materially clearer than an edge list |
| P2.9 | Convert HTML to semantic Markdown | 4 | 2 | B | M | The input is presentation-heavy web content and relevant semantics are checked |
| P2.10 | Remove HTML comments | 1 | 3 | C | L | Target harness strips them and they contain no GAP, provenance, or maintainer contract |
| P2.11 | Remove YAML frontmatter fields | 2 | 3 | C | L | The target consumer ignores them and no publishing/build workflow needs them |
| P2.12 | Remove a ToC | 1 | 2 | D | M | It is generated or the file is short and no consumer uses its anchors |
| P2.13 | Remove code comments | 2 | 3 | C | M | Types/code fully express intent and comments add no contract or gotcha |
| P2.14 | Shorten code examples | 2 | 3 | C | M | Omitted setup is conventional and the example’s purpose stays runnable or explicit |
| P2.15 | Replace negative wording with a positive action | 2 | 2 | C | H | The replacement fully specifies the safe alternative |
| P2.16 | Retain explicit prohibition | 3 | 1 | C | H | The forbidden action is tempting, dangerous, or ambiguous |
| P2.17 | One directive per line | 2 | 1 | C | H | Splitting does not duplicate scope or break a compound invariant |
| P2.18 | Telegraphic English and article removal | 2 | 2 | C | M | Meaning remains natural, unambiguous, and tested on target models |
| P2.19 | Reorder sections by urgency and task frequency | 3 | 2 | B | M | Precedence is unchanged and retrieval headings remain stable |
| P2.20 | Stable prefix/cache-aware layout | 2 | 2 | C | L | The provider caches that prefix and edits will not destabilize it |
| P2.21 | Split one file into references | 4 | 2 | A | M | Routing is reliable and each link says what/when |
| P2.22 | Use nested or path-scoped instruction files | 4 | 3 | A | L | The target harness’s discovery, precedence, and reload behavior are verified |
| P2.23 | Use an import directive | 2 | 3 | A | L | The harness supports it and whether loading is eager or lazy is known |
| P2.24 | Cross-file deduplication with a canonical pointer | 3 | 2 | B | M | The consumer can retrieve the source cheaply and needs little local context |
| P2.25 | Remove URL tracking parameters or normalize links | 1 | 2 | D | M | Identity and required analytics/signatures are proven unchanged |
| P2.26 | Replace or remove images | 2 | 3 | C | L | Their exact information is available textually and the harness is not relying on vision |
| P2.27 | Use `llms.txt` as a documentation index | 2 | 1 | C | M | The consumer discovers it; it supplements rather than replaces task routing |
| P2.28 | Add ownership or last-verified metadata | 2 | 1 | C | M | A real process maintains it; otherwise it becomes a new fossil |
| P2.29 | Keep a compact removal ledger | 1 | 2 | D | M | Audit need exceeds context and maintenance cost; prefer VCS otherwise |
| P2.30 | Use a human source plus generated compact view | 3 | 3 | C | M | Generation is deterministic or validated and canonical ownership is explicit |

### 5.4 P3 — experiments, not durable defaults

| ID | Technique | E | R | Ev | P | Experimental boundary |
|---|---|---:|---:|---|---|---|
| P3.1 | LLMLingua-family prompt compression | 3 | 3 | B | L | Transient prompts/RAG with task-specific evals; not governance files by default |
| P3.2 | Query-aware sentence or block selection | 3 | 3 | B | M | Query is known and the omitted source remains retrievable |
| P3.3 | LLM-generated session compaction | 3 | 3 | B | M | Structured artifact ledger, repeated-cycle tests, and source retention exist |
| P3.4 | Telegraph English / BabelTele | 3 | 4 | C | L | Ephemeral machine-only channel with fixed compressor-reader pair |
| P3.5 | MetaGlyph-style symbolic metalanguage | 3 | 4 | C | L | Selection/extraction microtask with per-operator fidelity tests |
| P3.6 | Context Codec or commitment-level atoms | 2 | 3 | C | L | Use the semantic model, not its notation, until independently validated |
| P3.7 | TOON, TRON, or another compact serializer | 2 | 4 | B | L | Structured tool I/O benchmarked against JSON for each target model |
| P3.8 | AST-based source compression | 2 | 4 | C | L | Read-only exploration; never replace exact source or contract |
| P3.9 | Reverse Chain-of-Density passes | 2 | 3 | D | M | Each pass is checked against the semantic ledger; stop on first loss |
| P3.10 | Ambient-knowledge ablation | 2 | 3 | C | L | Representative weaker models pass and the information remains discoverable |
| P3.11 | LLM compressor + independent reviewer | 3 | 3 | B | M | Reviewer has source, ledger, deterministic checks, and different failure incentives |
| P3.12 | Custom token-normalization choices | 1 | 3 | C | L | Actual target tokenizer shows a saving and readability does not fall |

### 5.5 Reject as automatic behavior

| ID | Practice | E | R | Why rejected |
|---|---|---:|---:|---|
| X1 | Invent missing commands, thresholds, conditions, or examples | 0 | 4 | Changes the operational contract |
| X2 | Delete or truncate by position, including “middle” content | 0 | 4 | Position is not semantic importance |
| X3 | Blanket-strip comments, frontmatter, metadata, ToCs, rationale, or examples | 1 | 4 | Each can carry contracts or routing |
| X4 | Base64-encode text or images to “compress” tokens | 0 | 4 | Usually expands tokens and destroys inspectability |
| X5 | Translate into Chinese, Classical Chinese, or another language solely for token savings | 1 | 4 | Model/tokenizer dependent; harms maintenance and exact terminology |
| X6 | Use emoji as semantic flags or replace modality with icons | 0 | 4 | Often multi-token, ambiguous, inaccessible, and non-portable |
| X7 | Introduce a private abbreviation dictionary, symbolic legend, or hash-only references | 1 | 4 | Adds decode state and catastrophic key-loss risk |
| X8 | Duplicate critical rules at both top and bottom | 1 | 3 | Consumes context and creates drift/precedence ambiguity |
| X9 | Treat `<details>`, visual collapse, or a link as automatic lazy loading | 0 | 4 | UI collapse does not imply context exclusion |
| X10 | Emit unsupported `load_if`, `priority`, `tokens`, `@include`, or transclusion syntax | 0 | 4 | Imaginary controls are inert or misleading |
| X11 | Enforce universal line, word, rule-count, or compression-ratio targets | 1 | 3 | Thresholds are harness/task heuristics, not fidelity criteria |
| X12 | Delete everything “discoverable in code” without pricing retrieval and ambiguity | 2 | 4 | Agents may not know what to search or which source is authoritative |
| X13 | Assume scripts, references, or images cost zero tokens | 0 | 3 | Loading and output behavior is harness-specific |
| X14 | Use lexical similarity or one LLM judge as proof of preservation | 0 | 4 | Misses rare but decisive constraints and shared blind spots |
| X15 | Create unrequested `removed.md`, `notes.md`, or `original.md` sidecars in runtime scope | 0 | 3 | Adds clutter and may itself be loaded; VCS is usually safer |

## 6. Safe transformation patterns

Apply transformations in this order:

> protect -> delete proven waste -> relocate conditional content -> rewrite -> formalize
> matching structures -> micro-optimize

Later stages have smaller expected gains and higher semantic risk
(`qwen.md `4`; `fable.md `4`; `codex.md `39`).

### 6.1 Prose to directive

Before:

> In situations where a generated file already exists, it is generally important that
> contributors avoid editing that file directly, because the generator is the source of
> truth.

After:

> If a generated file exists, edit its generator; do not edit the generated file.

Preserved atoms: trigger, object, safe action, prohibition, and rationale. Do not reduce
this to `Avoid generated files`; that loses the required alternative.

### 6.2 Default plus exception

Before:

> Use the fast parser for ordinary inputs. For signed inputs, the fast parser cannot
> validate the signature, so use the validating parser. Malformed input must stop the
> workflow.

After:

- Default: use the fast parser.
- Signed input: use the validating parser.
- Malformed input: stop.

This is safe only if the three branches are exhaustive or a fallback is stated.

### 6.3 Hoisted scope

Before:

- For files under `api/`, validate the schema before editing.
- For files under `api/`, preserve field order.
- For files under `api/`, run the compatibility check.

After:

> Under `api/`:
>
> - validate the schema before editing;
> - preserve field order;
> - run the compatibility check.

Never hoist across a heading or paragraph if that would widen the condition.

### 6.4 When a table is better

Use a table only if all rows answer the same questions. For example:

| Input state | Action | Stop condition |
|---|---|---|
| current | continue | none |
| stale | refresh | refresh fails |
| conflicting | escalate | human verdict |

Keep prose when rows need qualifications, sequence, nested exceptions, or long code.
Markdown table punctuation can cost more tokens than a short list
(`codex.md `16`; `qwen.md `6`).

### 6.5 When pseudocode is better

Pseudocode is useful for real control flow:

~~~text
if authority is unclear:
    report both rules
    stop before semantic deletion
elif source is superseded and verified:
    remove source payload
    retain provenance if needed
else:
    preserve
~~~

It is unsafe if it omits actor, exceptions, transaction boundaries, concurrency,
fallbacks, or errors. The cited pseudocode paper studies **training-time fine-tuning**,
not automatic rewriting of durable instructions for stock models
(`claude-opus.md `6 T4`; `ds_pro.md `V.5`).

### 6.6 When formal notation is better

Use a schema, grammar, regex, type, or formula when the original content is already
formal. Pair unfamiliar notation with a plain-language sentence. Do not translate policy
into symbols merely because symbols look shorter.

Safe:

~~~text
name := lowercase-alnum (lowercase-alnum | "-")* lowercase-alnum
~~~

Unsafe:

~~~text
U ∧ M -> X; ¬A => Q
~~~

The second form cannot be audited without a legend and hides modality and domain meaning.

### 6.7 Examples

Prefer a small basis set:

- one ordinary success;
- one boundary case;
- one expected failure;
- one combination that exposes precedence.

Delete near-duplicates only after mapping each example to the semantic atoms it covers.
If an example contains the only exact output, it is not an example; it is a contract.

### 6.8 Negative rules

Use a positive instruction when it completely replaces the negative:

> Instead of “Do not edit generated files,” write “Edit the generator, then regenerate.”

Keep the prohibition when the unsafe action is likely or costly:

> Edit the generator, then regenerate. Never patch the generated file.

The corpus’s universal “positive beats negative” and “pink elephant” claims are not
supported strongly enough to erase explicit safety boundaries
(`ds_pro.md `VI`; `glm52.md `12.3`; `qwen.md `5`).

### 6.9 Exact anchors

Keep commands and identifiers byte-exact unless the task explicitly authorizes their
change. Prose around them may shrink. Run before/after extraction for:

- inline code spans and fenced code;
- paths and globs;
- URLs and anchors;
- uppercase modality;
- numbers with units;
- version strings;
- environment and configuration keys;
- quoted error/output text.

An exact-anchor match is necessary but not sufficient: a preserved command under the
wrong condition is still a semantic loss.

### 6.10 Comments and metadata

Classify before removing:

- **runtime/harness metadata:** preserve if recognized;
- **publishing metadata:** preserve if another workflow consumes it;
- **ownership/staleness metadata:** preserve if maintained;
- **GAP, provenance, and maintainer comments:** preserve unless moved to an equally
  reliable channel;
- **pure decoration or generated navigation:** removable after verification.

Claude Code currently strips block-level HTML comments from injected `CLAUDE.md`
content, but that is a platform-specific optimization, not a general Markdown property.
Direct file reads still expose those comments.

## 7. Artifact and harness profiles

Compression policy must be selected by artifact role. Applying one Markdown minifier to
all of these files is a design error.

### 7.1 Role matrix

| Artifact | Loading pattern | Keep hot | Move or retrieve | Primary hazard |
|---|---|---|---|---|
| Root `AGENTS.md` / `CLAUDE.md` | Usually every relevant session | authority, scope, nonstandard commands, invariants, permissions, gotchas, completion criteria | tutorials, component-specific workflows, long references | auto-generated encyclopedia |
| Nested/path rule | Harness and path dependent | only rules unique to that scope | common policy stays canonical above | assumed precedence or lazy loading |
| `SKILL.md` | metadata at discovery; body on activation | trigger, workflow, gates, routing, critical invariants | deep reference, schemas, examples, deterministic scripts | bloated body or weak trigger |
| Memory/handoff | startup, retrieval, or compaction dependent | current goal/state, decisions, artifacts, risks, next action | raw history and closed investigations | narrative drift, stale policy |
| Reference guide | on demand if routing works | self-contained topic contract and anchors | unrelated topics | blind links and deep chains |
| Web documentation | fetched or indexed per query | answer, prerequisites, syntax, constraints, examples | chrome, navigation, scripts, styling | confusing presentation with content |
| Tool/API schema | passed with tool or fetched exactly | names, types, descriptions, required fields, errors | unrelated tools | shortening exact interface |
| Source/code example | read on demand | semantics needed for the lesson | repetitive boilerplate only if explicit | breaking runnable or contractual code |

### 7.2 Root agent instructions

A compact root file should answer:

1. What authority and scope does this file have?
2. What must the agent do differently in this repository?
3. Which exact commands or checks are non-obvious?
4. What actions are forbidden or require approval?
5. What gotchas cannot be discovered cheaply?
6. Where should task-specific detail be read, and when?
7. What proves the task is complete?

Do not turn it into a repository tour, dependency dump, style-linter manual, or catalog
of generic engineering advice (`codex.md `4, `38`; `claude-opus.md `4-5`).

### 7.3 Current Codex semantics

Current Codex documentation says project instructions are assembled from the project
root toward the **current working directory**, with closer instructions taking
precedence. It does not promise to discover every nested `AGENTS.md` merely because the
agent later edits a file below it. The default `project_doc_max_bytes` setting is
32 KiB. Therefore:

- nested `AGENTS.md` is conditional on launch/CWD topology, not a universal free
  retrieval tier;
- test discovery and precedence in the real surface;
- keep a root route or duplicate only the minimum safety-critical scope when a nested
  file is not guaranteed to load;
- treat the byte cap as a harness limit, not an optimal content target.

See the current [Codex `AGENTS.md` documentation](https://learn.chatgpt.com/docs/agent-configuration/agents-md)
and [configuration reference](https://learn.chatgpt.com/docs/config-file/config-reference).

### 7.4 Current Claude Code semantics

Current Claude Code behavior differs:

- ancestor `CLAUDE.md` files are concatenated at launch;
- descendant `CLAUDE.md` files under the working directory load when Claude reads files
  in those subdirectories;
- `@path` imports are eager, so splitting by import improves organization but not startup
  context;
- `.claude/rules/` can use supported `paths` frontmatter for conditional loading;
- block-level HTML comments are stripped from injected `CLAUDE.md`, but remain visible
  on a direct file read;
- Claude Code reads `CLAUDE.md`, not `AGENTS.md`, unless the latter is imported or
  linked;
- the documentation recommends a target below roughly 200 lines per `CLAUDE.md`. That
  is a platform recommendation and review trigger, not a universal correctness law.

See [How Claude remembers your project](https://code.claude.com/docs/en/memory).
Symlinks can work but have Windows privilege, portability, repository, and tool-behavior
costs; a verified import or generated synchronization may be safer.

### 7.5 Skills

For Agent Skills, discovery metadata and activated instructions have different costs.
The description must say **what the skill does and when to use it** using task vocabulary.
Once triggered, the whole `SKILL.md` body loads; references load when read, and scripts
contribute at least their invocation and output.

The [Agent Skills specification](https://agentskills.io/specification) recommends:

- required `name` and `description` frontmatter;
- a focused body, with deep content in `references/`;
- one-level file references;
- scripts that are self-contained and report useful errors;
- validation of the skill package.

The current OpenAI skill guidance similarly uses progressive disclosure, but its supported
metadata and catalog-budget behavior are not identical to every Agent Skills client. Do
not copy platform-specific frontmatter such as `load_if` unless the target documents it.
This repository imposes the stricter design budget: future `SKILL.md` must be at most
200 lines, references stay one level deep, and every route gets a what/when pitch.

### 7.6 Memory and handoffs

Memory should preserve decision-relevant state, not transcript chronology:

~~~markdown
## Goal
## Current state
## Decisions and reasons
## Artifacts changed or inspected
## Exact errors and unresolved risks
## Next actions
## Superseded
~~~

Preserve which facts came from users, tools, source files, or inference. Do not promote a
one-off observation into permanent policy. Repeated compaction must merge by decision and
artifact identity, not re-summarize the entire previous summary
(`codex.md `6, `25-26`; `mimo.md `1-3`).

### 7.7 Long references and web documentation

References must be internally sufficient for the task that retrieves them, but need not
repeat global policy. Use descriptive filenames, stable headings, and a ToC when the file
is long enough to benefit. Keep routing shallow.

HTML-to-Markdown conversion is a separate high-value preprocessing operation for web
pages. It can remove navigation, scripts, styles, and interactive markup while preserving
the main content. Cloudflare reports about a sevenfold token reduction for one real
documentation page, but that number is page- and converter-specific. Validate code,
tables, tab labels, JSON-LD, metadata, hidden primary content, and dynamic output before
discarding HTML. This does not prove that already-authored Markdown needs heavy syntax
minification.

### 7.8 Tool schemas, code, and structured contracts

Do not summarize away interface names, required fields, types, enum values, error
semantics, or exact output. Better levers are:

- expose only tools relevant to the task;
- remove duplicate descriptions after confirming the model still selects correctly;
- generate schemas from a canonical source;
- fetch large reference schemas on demand if the harness supports it;
- shorten examples around the contract, not the contract itself.

“Preserve all code byte-for-byte” is also too broad. A tutorial can omit conventional
boilerplate if it labels the omission; a runnable command, migration, schema, regex, or
example-as-spec cannot.

## 8. Canonical compression pipeline

The compressor should behave like a cautious compiler with reviewable intermediate
artifacts, not like a generic summarizer.

### Stage 0 — establish contract

Collect:

- files in scope and authoritative source order;
- artifact type and intended reader;
- target harnesses, models, tokenizers, and loading behavior;
- canonical human source vs canonical compact source;
- allowed mutations: rewrite only, split, add references, create scripts, or no new files;
- compression mode: safe, balanced, or experimental;
- required validation and user approval boundaries.

Stop if authority or requested scope is materially ambiguous.

### Stage 1 — inventory and baseline

Record per file:

- bytes, lines, headings, links, anchors, code fences, comments, and frontmatter;
- estimated and, when available, actual tokens for each target tokenizer;
- duplicate blocks and near-duplicate candidates;
- existing imports, nested instructions, skills, generated files, and enforcement;
- representative tasks and baseline outcomes.

Do not call bytes divided by a constant a tokenizer count. An estimate is acceptable for
ratios if labelled consistently.

### Stage 2 — build the semantic ledger

Extract `4.1` fields. Add literal inventories and map every example to the rules it
covers. Give disputed or uncertain atoms an explicit status. The ledger is the semantic
checksum used later (`codex.md `8, `21`).

### Stage 3 — build authority and behavior graphs

Create two conceptual graphs:

- **authority graph:** duplicate claims, imports, scopes, overrides, generated copies,
  and canonical sources;
- **behavior graph:** trigger -> rule -> action -> verifier -> failure/stop.

Flag conflicts, blind pointers, missing fallbacks, and rules that have no operational
effect.

### Stage 4 — classify each block

Use:

- **KEEP-HOT:** needed in nearly every relevant run or before an unknowable mistake;
- **KEEP-SCOPED:** needed for a predictable path, task, platform, or lifecycle event;
- **KEEP-ON-DEMAND:** detailed reference whose need is discoverable;
- **EXECUTE:** deterministic computation or enforcement belongs in a real mechanism;
- **COMPRESS:** semantics remain but wording/shape can shrink;
- **DELETE:** proven duplicate, stale, generic, or irrelevant;
- **REVIEW:** conflict, uncertain rationale, unsupported syntax, or high-loss proposal.

The classifier must cite the source section and reason for any DELETE or REVIEW decision.

### Stage 5 — delete proven waste

Apply only P1.1-P1.4 and safe normalization first. Re-run exact-anchor and ledger
coverage checks. This isolates high-confidence savings and keeps later judgment auditable.

### Stage 6 — relocate conditional material

Choose a verified mechanism: nested instruction, path rule, skill, reference, asset,
script, schema, or generated view. Add a what/when route at the consumer. Measure
**expected loaded cost**, including activation and retrieval.

If the target cannot reliably retrieve it, relocation is deletion in disguise.

### Stage 7 — rewrite preserved semantics

Apply condition-action, default-exception, canonical vocabulary, shared-scope hoisting,
closed lists, stop conditions, and example curation. Preserve literals separately.

Rewrite one semantic unit at a time. Do not repeatedly recompress already compressed
text without the original and ledger; that creates generational loss
(`claude-opus.md `8.8`).

### Stage 8 — select matching representations

Use tables, pseudocode, grammars, schemas, signatures, formulas, or diagrams only after
the suitability tests in `6. Test unfamiliar formats against plain Markdown. Never
formalize merely to hit a token target.

### Stage 9 — validate

Run the checks in `9. Any lost hard constraint, exact anchor, condition, exception,
precedence edge, or behavior is a failure, regardless of token savings.

### Stage 10 — report and gate

Report:

- files changed and structural moves;
- before/after bytes, lines, and target-tokenizer counts;
- expected startup/on-demand context change;
- deleted, relocated, rewritten, and disputed semantic units;
- exact-anchor and link differences;
- validation results and untested assumptions;
- residual risk and rollback path.

Require human approval before resolving disputed authority, deleting high-risk semantic
content, or adopting an experimental representation unless the user already authorized
that class of change.

### Compression modes

| Mode | Allowed by default | Excluded |
|---|---|---|
| **Safe** | proven duplicates, generic filler, verified stale content, surface normalization, low-risk wording | semantic relocation, example/rationale deletion, exotic formats |
| **Balanced** | Safe plus verified scoping, skill/reference splits, structured rewrites, curated examples | custom notation and model-specific prompt compression |
| **Experimental** | Balanced plus P3 techniques in isolated outputs with target evals | any irreversible replacement of the source |

Modes set allowed transformations, not promised ratios.

## 9. Validation and evaluation

### 9.1 Static preservation checks

| Check | Detects | Limitation |
|---|---|---|
| Exact-anchor diff | missing or changed commands, paths, IDs, URLs, versions, numbers | cannot detect wrong scope |
| Semantic-ledger coverage | omitted rule/fact/exception/rationale | depends on ledger quality |
| Modality comparison | weakened MUST/NEVER/ONLY semantics | synonyms need review |
| Condition/exception graph diff | broadened or narrowed rules | complex prose may require judgment |
| Heading/link/anchor check | broken routing and blind references | a valid target may still be irrelevant |
| Fence/frontmatter parse | malformed Markdown, YAML, or code blocks | syntax validity is not semantic fidelity |
| Duplicate/conflict scan | repeated and contradictory rules | similarity can over-report |
| Unsupported-control scan | imaginary frontmatter/import/DSL | requires a current harness profile |

### 9.2 Semantic probes

Ask both original and compressed documents:

- enumerate all hard prohibitions and approval gates;
- state each path/platform condition and exception;
- reproduce exact commands, identifiers, and output shapes;
- resolve representative precedence cases;
- identify when to stop, verify, retry, or escalate;
- explain the non-obvious rationale that constrains generalization;
- locate the correct on-demand reference from a task prompt.

Probe sets are coverage tools, not proof. Rotate them and trace every answer to the
ledger; otherwise a compressor can overfit the questions.

### 9.3 Behavioral differential evaluation

For representative tasks, compare fresh sessions under:

- **A — original documentation;**
- **B — compressed documentation;**
- **C — no documentation.**

Use the same harness, model version, reasoning setting, tools, sandbox, task, and grading
environment. Repeat enough runs to expose variance. The compressed version should beat
or match A on success and safety while reducing relevant cost; both should be compared
with C because a context file can be worse than no file.

Measure:

- task success and functional correctness;
- hard-rule compliance and unsafe actions;
- steps, tool calls, retries, and unnecessary exploration;
- startup, total input, cached input, output, and reasoning tokens when exposed;
- wall time and monetary cost;
- re-reads, re-fetches, and rediscovery;
- link/skill activation accuracy;
- human maintenance and audit time.

Primary metric:

`cost per successful, policy-compliant task`

Compression ratio is a diagnostic, not the objective.

### 9.4 Skill-specific evaluation

The future skill needs tests at three levels:

1. **Triggering:** description selects the skill for appropriate requests and not for
   adjacent summarization or source-minification tasks.
2. **Transformation:** every allowed operator preserves the ledger and exact anchors.
3. **Outcome:** compressed artifacts retain or improve representative agent behavior.

Include adversarial fixtures: comments carrying a GAP, YAML used by a renderer, an
example that is the only output contract, conflicting nested rules, eager imports,
Windows paths, signed URLs, and a command whose condition matters.

### 9.5 Reviewer and idempotency

An LLM reviewer should receive the original, candidate, semantic ledger, diff, and static
check results. It should try to find losses, not rate prose quality. A different model or
prompt reduces but does not remove correlated failure.

Run the compressor twice. A stable result should converge. Continued shrinkage usually
signals generational loss, unstable classification, or ratio chasing
(`claude-opus.md `9.4`; `qwen.md `4`).

### 9.6 Fail and rollback conditions

Fail the candidate if any of these occur:

- missing or weakened hard constraint;
- changed scope, trigger, exception, precedence, default, or stop condition;
- changed exact anchor without explicit authorization;
- invented operational fact;
- broken or misleading route;
- unsupported harness syntax presented as functional;
- task-success or safety regression beyond the predeclared tolerance;
- savings depend on hiding the original without a recovery path.

## 10. Smells and detectors

The first six names come from the configuration-smell study discussed in
`claude-opus.md `1.2`. That study establishes that the smells occur; it does not prove a
universal performance penalty or a magic file-size threshold.

| Smell | Detection lead | Safe response |
|---|---|---|
| **Context bloat** | high always-loaded size; many low-frequency sections | classify hot/scoped/on-demand; measure expected residency |
| **Lint leakage** | prose duplicates formatter/linter rules | retain tool invocation and non-obvious exceptions; remove duplicated payload after verification |
| **Skill leakage** | long task-specific procedure in global instructions | move to a skill; keep trigger and critical gate |
| **Blind reference** | path or URL without what/when wording | add a retrieval pitch; validate target and anchor |
| **Initialization fossil** | generated inventory, mutable dependency list, or stale architecture snapshot | replace with current invariant, query, or canonical source |
| **Instruction conflict** | opposing modalities or overlapping scopes | record both, determine authority, never silently pick |
| **Scope leakage** | frontend/backend/platform rule loaded everywhere | use verified path/task scoping |
| **Duplicate payload** | same semantic atom in several files | choose one canonical home; preserve minimal consumer context |
| **Stale mutable fact** | version/date/status without owner or refresh path | verify, remove, or attach maintained provenance |
| **Unsupported control** | unknown YAML keys, `@include`, HTML import, custom priority syntax | remove claim of function or target a documented harness |
| **Lexical cryptography** | many abbreviations, symbols, IDs, or a legend needed to read rules | restore canonical natural language |
| **Example drift** | example contradicts prose or current code | treat as conflict; update only from authority |
| **Rationale orphan** | “because” text with no active rule | delete if historical; attach if it constrains interpretation |
| **Rule orphan** | directive has no scope, actor, trigger, or verifier | complete from source or mark GAP; do not guess |
| **Routing depth** | reference points to another index before useful content | flatten to one-level, pitched routes |
| **Metric theater** | impressive token ratio without task evaluation | report cost per successful task and untested risk |

Detectors should produce candidates, evidence, and confidence. They must not delete
automatically simply because a regex matched words such as “note,” “overview,” or
“example.”

## 11. Evidence assessment

All numbers below were checked against the linked primary source for this independent
phase. They are study-specific observations, not universal targets. The corpus’s
unverified point estimates are not repeated as general facts.

### 11.1 Directly relevant evidence

| Source | Verified finding | What it supports | Important limitation | Grade |
|---|---|---|---|---|
| [Evaluating AGENTS.md](https://arxiv.org/abs/2602.11988) | Across roughly 140 Python issues plus a second coding benchmark, LLM-generated context files did not improve success over no file, while developer-written files were directionally better; both added steps and roughly one-fifth more cost | Context files can hurt; provenance and behavioral A/B/C matter | Python coding tasks; small success differences; security and other outcomes not tested | A |
| [SkillReducer](https://arxiv.org/abs/2603.29919) | Structure-aware compression reduced skill bodies by roughly two-fifths and improved the study’s aggregate score, but about one in seven skills regressed; moving examples was a recurring failure | Semantic taxonomy, feedback loop, progressive disclosure, example protection | Anthropic protocol, high benchmark ceiling, other skill clients untested | A |
| [Configuration Smells in Agent Context Files](https://arxiv.org/abs/2606.15828) | A study of about 100 popular repositories operationalized six recurring smells; conflict detection had materially lower precision than simpler detectors | Useful lint candidates and the need for review confidence | Observational prevalence, small grey-literature base, no causal performance test | B |
| [Agent READMEs efficiency study](https://arxiv.org/abs/2601.20404) | In paired repository tasks, runtime and output tokens fell, but median total tokens slightly increased | Efficiency metrics must separate input/output and report distributions | Functional equivalence was not comprehensively graded | B |

The first study does **not** establish a 150-line or 40-instruction optimum. Its ablations
did not find a strong length effect, and its developer-written gains were not a universal
significance result. The smell study’s 200-line threshold is a borrowed heuristic, not a
causal boundary.

### 11.2 Attention, count, and format evidence

| Source | Verified finding | Safe inference | Invalid inference | Grade |
|---|---|---|---|---|
| [Prompt Design at Scale](https://arxiv.org/abs/2607.19257) | Five models on one synthetic corpus lost perfect compliance by around 80 simultaneous rules; format ranking and placement effects were model-specific | Keep active instructions few, specific, and target-tested | Every file fails at 80 rules; Markdown is always best/worst | B |
| [Lost in the Middle](https://arxiv.org/abs/2307.03172) | Relevant information position affected long-context QA and key-value retrieval | Front-load critical authority and make structure searchable | Delete the middle or duplicate every rule at both ends | B |
| [IFScale](https://arxiv.org/abs/2507.11538) | Models degraded on a synthetic many-instruction inclusion task and showed primacy effects | Instruction count is a review signal | Universal cap of 40, 80, or any other point | B |
| [Context Rot](https://research.trychroma.com/context-rot) | Controlled simple tasks showed degradation with length, distractors, and similarity across many models | Minimize irrelevant context and evaluate retrieval | A fixed safe context length for all tasks/models | B |
| [Prompt-format sensitivity](https://arxiv.org/abs/2310.11324) | Meaning-preserving format changes caused large, model-specific variance in few-shot classification | Test plausible formats on target models | One benchmark proves tables, Markdown, or YAML universally superior | B |
| [Structured Context](https://arxiv.org/abs/2602.05447) | In a SQL-generation proxy, common formats had no aggregate accuracy winner; compact unfamiliar formats could increase generated reasoning tokens | Familiarity and downstream cost matter | Raw input-token count determines best format | B |

These studies support **format humility**. They do not license a single Markdown style
guide to claim mechanistic superiority for tables, arrows, sparse punctuation, or any
other surface form.

### 11.3 Compact notation and prompt compression

| Source | Verified finding | Relevance to durable Markdown | Grade |
|---|---|---|---|
| [Notation Matters](https://arxiv.org/abs/2605.29676) | TOON/TRON saved up to roughly one-quarter of tokens in agentic tool exchange but incurred about 9-14 percentage-point accuracy penalties and parsing failures | Strong warning against replacing familiar structures without end-to-end tests | B |
| [LLMLingua](https://arxiv.org/abs/2310.05736), [LongLLMLingua](https://arxiv.org/abs/2310.06839), [LLMLingua-2](https://arxiv.org/abs/2403.12968) | Learned prompt compressors can preserve task performance at substantial ratios in selected prompt/RAG settings | P3 transient-prompt option, not a default editor for policy files | B |
| [Prompt Compression in the Wild](https://arxiv.org/abs/2604.02985) | End-to-end speed gains appeared only in matched prompt/model/hardware regimes; preprocessing can erase the benefit | Count compressor overhead and hardware, not only input tokens | B |
| [Telegraph English](https://arxiv.org/abs/2605.04426) | A compact representation retained high factual recovery in a QA experiment | Interesting machine-only experiment | C |
| [BabelTele](https://arxiv.org/abs/2606.19857) | A preprint reports dense model-readable representations, with performance dependent on compressor-reader pair and task | Supports experimentation, not shared durable governance | C |
| [MetaGlyph symbolic instructions](https://arxiv.org/abs/2601.07354) | Large token reduction coexisted with highly variable operator fidelity, including poor results for several models/operators | Direct evidence that a symbol can parse yet fail semantically | C |
| [Context Codec Language](https://arxiv.org/abs/2605.17304) | A small diagnostic motivates preserving commitment-level atoms | Its semantic decomposition is useful; its notation remains immature | C |

SkillReducer is particularly relevant here: generic LLMLingua-style compression retained
less skill behavior than the structure-aware method. A method good for retrieved prose is
not automatically good for instructions.

### 11.4 Adjacent evidence: code, pseudocode, and compaction

| Source | Verified finding | Transfer boundary | Grade |
|---|---|---|---|
| [Training with Pseudo-Code for Instruction Following](https://arxiv.org/abs/2505.18011) | Fine-tuning models on paired pseudocode improved instruction benchmarks | Training-time result; not evidence for blind inference-time doc conversion | C |
| [Code formatting and token efficiency](https://arxiv.org/abs/2508.13666) | Removing formatting reduced inputs in fill-in-the-middle code completion | Code-completion setting; does not justify minifying human-maintained docs or contracts | C |
| [DETAIL Matters](https://arxiv.org/abs/2512.02246) | More specific prompts improved a small reasoning study | Supports specificity; does not support the dossiers’ claimed large penalty for one abbreviation | C |
| [Context Compaction Theory](https://arxiv.org/abs/2608.01326) | Formalizes selection vs generated summaries and relates achievable budget to communication complexity | Strong conceptual lower bound; preliminary and not a Markdown evaluation | C |
| [Rate-Distortion View of Memory Compaction](https://arxiv.org/abs/2607.08032) | Survey argues query-unknown, irreversible pruning repeatedly loses later-needed information | Supports reversibility and decision-aware retention | C |
| [Remember the Decision, Not the Description](https://arxiv.org/abs/2605.10870) | Decision-centric memory outperformed descriptive criteria in its evaluated agents | Useful for memory profile, not all documentation | B |
| [Self-Compacting Language Model Agents](https://arxiv.org/abs/2606.23525) | Task-boundary-aware compaction beat fixed-interval baselines in evaluated long agents | Supports semantic boundaries for sessions; not static file deletion | B |
| [Parallel Context Compaction](https://arxiv.org/abs/2605.23296) | LLM summary size and retention varied; block compaction gave more control | Supports structured, measured session summaries | C |

### 11.5 Platform and practitioner evidence

| Source | What is authoritative or useful | Limitation | Grade |
|---|---|---|---|
| [OpenAI Codex `AGENTS.md`](https://learn.chatgpt.com/docs/agent-configuration/agents-md) and [skills](https://learn.chatgpt.com/docs/build-skills) | Current discovery, precedence, byte-budget, metadata, and progressive-disclosure behavior | Product behavior can change; applies to Codex surfaces | A |
| [Claude Code memory](https://code.claude.com/docs/en/memory) and [Agent Skills spec](https://agentskills.io/specification) | Current eager imports, nested/path rules, comment handling, skill layout, and limits | Claude-specific behavior is not Codex behavior | A |
| [OpenAI current model guidance](https://developers.openai.com/api/docs/guides/latest-model) | Leaner prompts improved an internal coding-agent sample; guidance says ablate one group at a time and retain requirement-bearing examples | Internal sample and current model family; directional ranges only | B |
| [Anthropic context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) | Smallest high-signal set, just-in-time retrieval, canonical examples, compaction, and retrieval-cost tradeoff | Official practice guidance, not a controlled Markdown trial | B |
| [Cloudflare AI consumability](https://developers.cloudflare.com/style-guide/how-we-docs/ai-consumability/) | Real HTML-to-Markdown pipeline and a page-level token comparison | Web conversion, not intrinsic Markdown compression | B |
| [GitHub’s review of 2,500+ agent files](https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/) | Useful descriptive patterns for role, commands, layout, and boundaries | Repository survey/blog; does not establish causal performance | C |
| `available_skills.md `1-5` prior art | Useful implementation ideas, especially tokens-per-task, structured summaries, and honest workload-specific measurement | Self-reported benchmarks, popularity drift, and uneven provenance | C-D |

### 11.6 Claims not accepted

The following recurring corpus claims were not substantiated strongly enough:

- a specific abbreviation such as “auth” causes a roughly 30-point accuracy loss;
- Markdown tables are generally three to five times more token-efficient;
- a fixed number of rules, lines, words, or tokens is universally optimal;
- generated agent files are always harmful or developer files always beneficial;
- HTML comments, frontmatter, or scripts are universally zero-token;
- diagrams/images are invisible to agents;
- positive wording always outperforms explicit prohibitions;
- repeating critical rules at both ends reliably beats single canonical placement;
- token density in another human language reliably improves cross-model instruction
  following;
- any fixed 70%, 80%, 90%, or 95% compression target preserves behavior.

Treat these as hypotheses only if a target-specific evaluation justifies them.

## 12. Dossier-by-dossier critical audit

This section records what each source contributed and what was filtered out. Agreement is
not counted by repetition.

| Source | Best contribution | Main correction or rejection |
|---|---|---|
| `claude-opus.md `1-12` | Concrete smell taxonomy, block classification, pitched links, conflict graph, checks, and a complete pipeline | Several “delete-on-sight” rules are role-blind; tail duplication and arbitrary budgets are weak. Its `13` worked example is **not compression**: it invents `pytest tests/unit -q`, a Docker command, a four-minute duration, and `api/` / `db/` gates absent from the source |
| `codex.md `8, `19-22, `29-40` | Strongest conceptual treatment: semantic IR, ablation, expected residency, semantic checksum, discoverability tax, behavior-aware metrics | Some weights, ratios, and stage budgets are illustrative heuristics. “Discoverable” information still needs retrieval-cost and ambiguity tests |
| `ds_flash.md `2-7` | Correct priority on representation, delete/relocate/rewrite order, progressive disclosure, and abbreviation caution | Calls destructive operations lossless; proposes unsupported metadata and fragile line-number routing |
| `ds_pro.md `II-VI` | Useful staged audit and explicit must-preserve list | Its lossless catalog includes semantic deletions; Telegraph English, dictionary backreferences, and toolchain-first absolutism are unsafe defaults |
| `fable.md `3-8` | Clear compression ladder, tiered skill architecture, and executable-validation emphasis | Treats comments/frontmatter as broadly disposable and scripts as zero-token; several numerical claims are unsupported |
| `gemini.md `2-7` | Emphasizes progressive disclosure, verification, and operational structure | Universal table conversion, predicate-logic shorthand, a verifier for every rule, memory purging, and “slot capacity” thresholds would add ambiguity or invented process |
| `glm52.md `4-12` | Protected-facts pattern, faithfulness guard, hybrid-source idea, and measurable forms | Position-aware truncation, sandwich duplication, comment/frontmatter stripping, emoji, and hard SDE targets are not safe |
| `grok.md `3-7` | Sensible safe/lossy separation, compressor-reviewer loop, and reviewer checklist | Byte-preserving all code is overbroad; backup sidecars and unsupported frontmatter need target-specific justification |
| `hy3.md `4-8` and `10-11` | Recognizes rate-distortion, negative constraints, and integrity risks | Contains the corpus’s most dangerous token hacks: base64, CJK translation, emoji flags, invented transclusion/frontmatter, and AST/prose deletion. Reject these defaults |
| `kimi.md `3-8` | Strong GAP/no-invention rule, fact inventory, example awareness, and staged verification | Comments-as-free, symlinks, repeated poles, translation tricks, and fixed hard limits are harness- or model-specific |
| `m3.md `3-9` | Useful distinctions among lexical, structural, referential, and semantic compression; notes cache stability and overcompression | Private shorthand, symbolic logic, hash IDs, imagined imports, top/bottom duplication, and graveyard sidecars create decode and drift risk |
| `mimo.md `1-10` | Rate-distortion and decision-value framing; reversibility and runtime-vs-static distinctions | Internally mixes passive preload with retrieval, and recommends dictionaries, position deletion, and blanket frontmatter stripping |
| `nvidia.md `2-9` | Good judgment matrix, two-stage validation, evaluation metrics, and anchor-contract idea | Mislabels many semantic deletions lossless, assumes images/cross-references are ignored, and imports prompt-compression claims too directly |
| `qwen.md `4-12` | Most balanced practical sequence after `codex.md`; explicitly recognizes contradictory format evidence and warns about abbreviation dictionaries | Fixed ceilings, tail repetition, and positive-only wording remain heuristics, not defaults |
| `spark1.md `4-9` | Correctly makes progressive disclosure central and preserves commands/gotchas | Telegraph triples, ID DSLs, abbreviation dictionaries, symlink synchronization, and aggressive ratio targets optimize apparent density over robustness |
| `available_skills.md `1-5` | Useful map of caveman compression, context-engineering skills, SkillReducer, and extreme baselines | Repository popularity and self-reported ratios are time-sensitive; output-brevity tools are not equivalent to input-policy compression |
| `reasearch_links.md` | Useful raw discovery queue | A URL is not evidence. The queue mixes primary papers, product docs, blogs, repositories, and marketing and must never be cited as a finding |

### 12.1 Cross-source disagreements and independent verdicts

| Disagreement | Position A | Position B | Independent verdict |
|---|---|---|---|
| Root context vs retrieval | preload everything important | root is only an index | keep unknowable high-cost gotchas hot; retrieve predictable detail |
| Examples | densest behavioral specification | expensive payload to move/delete | retain a minimal coverage basis; map each example to atoms |
| Rationale | necessary for generalization | prose bloat | keep consequence-bearing rationale; remove restatement/history |
| Negative rules | explicit NEVER is salient | “pink elephant” makes negatives harmful | give safe alternative and retain prohibition for cliffs |
| Tables | structured and compact | punctuation-heavy and model-sensitive | use only for repeated schemas/matrices |
| Pseudocode | clearer branching | loses nuance and evidence is training-time | conditional on complete control-flow preservation |
| Frontmatter/comments | metadata must survive | strip as zero-value | classify by consumer and role |
| Code | preserve byte-exact | strip comments/format/boilerplate | preserve contracts/runnable examples; shorten pedagogical code only with declared omissions |
| Duplicate at both ends | mitigates position bias | wastes context and drifts | one canonical placement, strong heading, optional true enforcement |
| Custom notation | large token savings | brittle, hard to maintain | P3 machine-only experiment |
| Human vs LLM source | dual corpus protects readability | one canonical source prevents drift | choose one authority; generate/validate the other if dual views are required |
| Fixed budgets | operational discipline | arbitrary across tasks/models | use as review triggers or repo mandates, never fidelity goals |

## 13. Worst practices: reject by default

These are ranked by expected damage, not by how strange they look.

### 1. Fabricating a better contract

Adding a missing command, verifier, scope gate, threshold, time estimate, or exception is
authoring, not compression. It can be valuable only as a separately reviewed proposal.
The `claude-opus.md `13` example demonstrates this failure while calling the additions
“defensible.” A compressor must instead mark the gap.

### 2. Silently resolving contradictions

A polished document with the wrong winner is worse than a verbose document that exposes
the conflict. Authority, scope, date, and provenance decide; confidence or majority vote
does not.

### 3. Position-based deletion or truncation

Recency, attention, and “lost in the middle” are weak importance proxies. Never delete
the middle of a document, the body of code, old memory, or low-attention blocks without a
semantic test. Later queries often need exactly what query-unknown pruning removed.

### 4. Format-blind “lossless” stripping

Comments can hold a GAP. Frontmatter can drive rendering or skill discovery. A ToC can
be a routing structure. Code comments can encode invariants. Metadata can establish
ownership or freshness. Examples can be the only specification. Classify roles first.

### 5. Pretending organization is lazy loading

A separate file, import, link, `<details>` block, image, nested `AGENTS.md`, or
frontmatter condition saves context only when the target harness actually excludes and
later retrieves it. Verify load traces where possible.

### 6. Minifying contracts and interfaces

Do not shorten tool names, schema keys, enum values, exact output, errors, commands,
paths, globs, or runnable code to meet a ratio. Reduce the surrounding explanation or
exposed tool set instead.

### 7. Lexical cryptography

Private abbreviations, symbol dictionaries, emoji flags, hash IDs, CJK translation,
custom DSLs, and dense tuple notation move tokens from payload into decoding state. Loss
of one legend entry can corrupt every reference. They also make review and cross-model
transfer worse.

### 8. Ratio chasing

A target such as “remove 70%” rewards deletion after useful savings are exhausted.
Likewise, “under 150 lines,” “under 40 rules,” or “15 words per sentence” can be a smell
threshold but not a success condition. Stop when marginal savings threaten utility.

### 9. Deleting all discoverable information

Code, README files, tests, and configuration may contain the same fact, but the agent
must know what to search, pay the retrieval cost, and choose the correct authority.
Retain the compact route, exact nonstandard command, or gotcha when discovery would be
unreliable.

### 10. Duplicating rules to exploit attention

Top-and-bottom repetition, copied policies in every subtree, and human/LLM twins without
generation controls create contradiction and cache churn. Prefer canonical placement,
good headings, scoped enforcement, and behavioral evaluation.

### 11. One-shot summarization with cosmetic validation

A fluent summary plus a high embedding score can omit one destructive-action boundary.
Use the ledger, exact-anchor checks, condition graphs, probes, and behavioral tasks.

### 12. Adding graveyards to active context

`removed.md`, `AGENTS.notes.md`, and `original.md` files are sometimes useful audit
artifacts, but creating them automatically can increase the very context and ambiguity
being reduced. Prefer version control and an external report unless the user requests a
dual corpus.

## 14. Implications for the future skill

This document is intentionally a long research foundation. The runtime skill must not
copy it wholesale.

### 14.1 Recommended package shape

~~~text
skills/compress-llm-documentation/
  SKILL.md
  references/
    preservation-contract.md
    rule-catalog.md
    artifact-profiles.md
    validation.md
    evidence-notes.md
  scripts/
    inventory.py
    extract-anchors.py
    check-links.py
    compare-ledger.py
    measure-tokens.py
  examples/
    agents-before.md
    agents-after.md
    skill-before.md
    skill-after.md
    memory-before.md
    memory-after.md
~~~

Names are proposed design, not created files. Keep references one level deep. Add a ToC
to any reference over 100 lines. Every `SKILL.md` route must state what the target
contains and when to read it.

### 14.2 What belongs in `SKILL.md`

The body, capped at 200 lines by this repository, should contain only:

- scope and trigger disambiguation;
- default safe mode and how explicit user requests select stronger modes;
- non-negotiable preservation rules;
- the stage sequence and stop gates;
- routing to artifact-specific and validation references;
- required report shape;
- explicit anti-goals.

Do not embed the evidence review, full technique catalog, large examples, or platform
tables in the body.

The description should trigger on requests to compress, minify, restructure, optimize,
deduplicate, or reduce context in LLM-facing Markdown. It should not trigger for ordinary
article summarization, source-code minification, or transient prompt compression unless
the user explicitly connects those tasks.

### 14.3 Deterministic scripts

Scripts should perform facts, not judgment:

- inventory Markdown structure;
- count bytes/lines and optional tokenizer outputs;
- extract anchors and modality candidates;
- detect duplicate candidates and unsupported controls;
- validate internal paths, headings, and fences;
- compare before/after literal sets and ledger records;
- emit machine-readable findings with useful errors.

They should not delete prose or decide that a rationale/example is irrelevant. Script
execution is not intrinsically zero-context: invocation, errors, and output still cost
context, and some agents may read source before running it.

No stack is adopted here because the target harness decision remains open. Python 3.13
with the standard library is the simplest candidate for deterministic cross-platform
analysis in this repository, but the choice must be recorded before implementation.

### 14.4 Skill workflow

The runtime workflow should:

1. inspect target files and local instructions;
2. identify artifact/harness profile;
3. produce or update the semantic ledger;
4. classify blocks with evidence;
5. propose structural moves and semantic-risk gates;
6. apply only user-authorized mode transformations;
7. run deterministic and semantic checks;
8. report savings, expected residency, risks, and untested assumptions.

For an audit-only request, stop after the proposal and findings. For an authorized edit,
make reviewable changes and validate them. The skill must never infer permission to split
files, create scripts, or adopt experimental notation from a request that asks only for
analysis.

### 14.5 Output contract

Every run should return:

~~~markdown
## Result
- mode:
- files analyzed / changed:
- bytes and target tokens before -> after:
- expected always-loaded context before -> after:

## Semantic accounting
- preserved:
- deleted as proven waste:
- relocated:
- rewritten:
- conflicts / gaps:

## Verification
- exact anchors:
- conditions / exceptions / precedence:
- links / harness controls:
- behavioral evaluation:

## Risk
- residual risk:
- unverified assumptions:
- rollback:
~~~

If actual tokenizer or behavioral data is unavailable, say so. Do not replace it with a
precise-looking estimate.

### 14.6 Evaluation corpus

Use examples as eval fixtures rather than decoration. The initial corpus should cover:

- compact and bloated root instructions;
- nested Codex and Claude scopes with different load behavior;
- a skill whose examples encode its output contract;
- memory with decisions mixed into chronology;
- a reference guide with blind links and duplicated policy;
- HTML converted to Markdown;
- malicious or accidental prompt injection inside source documentation;
- comments/frontmatter that are meaningful and meaningless;
- Windows paths, Unicode, signed URLs, code fences, tables, and malformed Markdown.

Each fixture needs a semantic ledger and behavioral questions. Include a no-document
baseline for agent tasks.

### 14.7 Provisional product principles

The future skill should:

- default to Safe mode;
- present deletion and relocation evidence, not just a rewritten file;
- distinguish current facts from proposed improvements;
- optimize expected context and cost per success;
- refuse automatic conflict resolution and unsupported control syntax;
- make experimental representations opt-in;
- keep original sources recoverable until validation passes;
- be able to say “no worthwhile compression remains.”

## 15. Open decisions

These decisions still gate implementation.

### 15.1 Target harness set

Choose one:

- **Codex first:** optimize `AGENTS.md` and current Codex skills; simplest semantics.
- **Claude Code first:** optimize `CLAUDE.md`, path rules, imports, and Claude skills.
- **Portable core plus adapters:** common semantic engine with explicit Codex/Claude/
  Cursor/Copilot profiles.

**Provisional recommendation:** portable semantic core plus Codex and Claude adapters.
The documents show that pretending their scoping semantics are identical is unsafe. Do
not add other harnesses until their official load behavior and eval tasks exist.

### 15.2 Evidence publication gate

Load-bearing claims in this foundation were independently checked. The original corpus
still contains citations and numerical claims that were not all validated. Before
publishing an evidence reference:

- resolve every cited identifier;
- separate peer-reviewed work, preprints, product docs, blogs, and self-reported tools;
- record version/date and exact transfer boundary;
- remove any statistic that cannot be traced to a primary source.

No runtime rule should depend on an unresolved citation.

### 15.3 Canonical source strategy

Decide whether the compact document is:

- the canonical human- and agent-maintained source;
- a generated view of a richer source;
- an agent-only artifact paired with a human source.

The second and third options require generation, drift detection, and clear authority.
The first requires readable compression rather than model-only notation.

### 15.4 Mutation and approval policy

Specify whether the skill may:

- edit in place;
- create references, scripts, or examples;
- move content across files;
- delete stale/historical material;
- change exact literals;
- resolve conflicts;
- run behavioral evals that incur API cost.

Recommended default: analysis plus Safe in-place edits when asked to compress; explicit
approval for cross-file architecture, semantic deletion above R1, paid evals, or P3
representations.

### 15.5 Evaluation target

Choose:

- representative repository tasks;
- target model/version/reasoning settings;
- target tokenizer(s);
- acceptable variance and regression threshold;
- security/destructive-action scenarios;
- maintenance and reviewer-effort measures.

Without this, “better” can only mean smaller.

### 15.6 Stack and dependencies

After the harness decision, record the implementation stack. A low-dependency Python
toolchain is the leading candidate because Python 3.13 and ripgrep are available, but
real tokenizer counts require adding and pinning a tokenizer library. Do not imply
`bytes / 3.9` is an absolute token budget.

### 15.7 Platform drift

Official load behavior, frontmatter fields, context budgets, and skill discovery can
change. The future skill needs versioned platform profiles or a “verify current docs”
step for structural changes. Unsupported or unknown clients should fall back to one
portable Markdown file plus explicit pointers, not guessed magic syntax.

## 16. Reconciliation with the prior derived documents

Sections 1-15 were frozen before this comparison. The pre-comparison file SHA-256 was
`4AB34EDA1A6680DC6E4DC98552208759289EFD78DE65D0FBF3A2F3D9AD6C7E91`.
Only then were `INDEX.md`, `EVIDENCE.md`, and `SYNTHESIS.md` read.

### 16.1 Agreement

The independent result agrees with the earlier synthesis on the durable core:

- optimize task utility and context placement, not prose aesthetics;
- delete proven waste, then relocate, then rewrite;
- use progressive disclosure where the harness truly supports it;
- inventory exact anchors and preserve hard constraints;
- make links descriptive and shallow;
- use Markdown as the portable default;
- treat memory as compact state and decisions rather than a transcript;
- reject private symbolic notation, CJK token hacks, and emoji tags by default;
- require staged validation, idempotency, and behavioral comparison;
- never invent facts or silently resolve conflicts.

This agreement increases confidence in the **direction**. It does not validate the
numerical ratios repeated across model-authored dossiers.

### 16.2 Earlier verdicts changed by the independent audit

| Prior derived position | Independent verdict | Reason |
|---|---|---|
| T3 scripts cost “0 tokens” (`SYNTHESIS.md `2.2`) | Scripts reduce prose residency but invocation, output, errors, and sometimes source inspection still cost context | Official skill docs describe on-demand execution, not literal zero total cost |
| Root <=150 lines and skill <=500 lines are universal tier budgets | Treat these as platform guidance or local mandates; use behavioral and load budgets | Current Codex has a byte cap; Claude and Agent Skills publish recommendations; tasks vary |
| Delete what the base model already knows (`2.4`) | Delete generic advice; ablate project facts only when target models know them and retrieval remains reliable | Model knowledge, versions, authority, and weaker-model behavior vary |
| Structured forms beat prose for the same content (`2.8`) | Use them only for a matching schema, branch structure, grammar, or quantitative relation | Controlled format work shows model-specific rankings; tables and notation can add tokens or errors |
| Repeat the top three rules at the tail (`3.3`) | Do not duplicate by default; use one canonical block, strong placement, and real enforcement | Lost-in-the-middle evidence does not test tail duplication; copies drift and consume attention |
| Any new identifier is hallucination (`2.3`) | Existing contractual literals need exact preservation; explicitly authorized new reference paths or generated artifacts are legitimate | Architectural refactoring necessarily creates some reviewed identifiers |
| Comments are free human notes | Only current Claude Code strips block HTML comments from injected `CLAUDE.md`; other readers and direct reads expose them | Harness-specific behavior was verified, not a Markdown property |
| A well-formed compact context file is already proven better than no file (`4.1`) | This is the project hypothesis to test, not a general empirical fact | The efficiency study did not comprehensively establish functional equivalence; the controlled AGENTS study found small/non-significant success differences |
| Nested rules provide a universal conditional tier | Model each harness separately | Current Codex discovery follows root to CWD; Claude can load descendant `CLAUDE.md` on file access and supports path rules |
| Images are broadly wrong for machine docs (`3.6`) | Keep critical text textual, but evaluate images by information role and multimodal harness | Some tasks and current agents intentionally consume images |
| Removal ledger should always be emitted (`5, `6`) | Prefer VCS and an external report; create an unloaded ledger only for a real audit/anti-ratchet need | Sidecars can become stale, ambiguous, or accidentally loaded |

The most serious source-level correction is `claude-opus.md `13`. Its “after” example
adds commands, a timing claim, and path conditions that the “before” text never states.
That is an authored policy proposal presented as compression. It violates the corpus’s
own no-invention rule and must not seed an eval as a positive example.

### 16.3 Useful refinements recovered from the derived documents

The earlier synthesis made several independently read ideas easier to see as named
operators. They are adopted with narrower boundaries:

- **Co-access partitioning:** if traces exist, group content by probability of being
  needed in the same task rather than by topic alone. This operationalizes P1.16.
- **State -> query:** replace mutable snapshots with a command or pointer to the
  canonical source when retrieval is cheap and unambiguous.
- **Docs-as-tests:** validate paths, commands, and schemas through real checks when those
  checks already exist or can be added with authorization. Do not invent a verifier for
  every rule.
- **Unhobbling review:** flag defensive rules written for older models or harnesses as
  stale candidates; never delete them merely because a newer model seems capable.
- **Rejected alternatives in memory:** preserve decision, rejected option, and reason
  when repeating the investigation would be costly.
- **Hybrid view:** a richer canonical source plus a generated compact view is safe only
  with deterministic generation or full ledger validation.

These refine the pipeline without changing its central rankings.

### 16.4 Evidence-ledger check

The earlier `EVIDENCE.md` correctly warned that every identifier was unverified. The
independent pass resolved all 41 arXiv IDs:

- 38 resolve to work matching the cited topic;
- three resolve but are **wrong citations**, not missing papers:
  `2310.11333` is strawberry orientation, not prompt-format sensitivity;
  `2312.00059` is ion-trap physics, not Prompt Cache;
  `2404.11576` is video prediction, not LongLLMLingua;
- the correct prompt-format ID is `2310.11324`;
- the correct LongLLMLingua ID is `2310.06839`.

Resolution does not validate every number attributed to a paper. Detailed claim checks
were prioritized for the evidence that changes the rule catalog; other papers remain
scope-limited by their actual abstracts and methods.

### 16.5 Final self-check

The comparison caused **no reversal** of the independent core. It:

- strengthened the rejection of fixed budgets, blanket “lossless” rules, and tail
  duplication;
- added named refinements for trace-based partitioning, mutable-state queries, and
  rejected-alternative memory;
- confirmed that the main remaining blocker is product scope and evaluation design, not
  a missing compression trick.

The future skill should use ``1-15 as the normative basis and this section as the audit
trail explaining where it differs from the earlier synthesis.

## 17. Primary references

### Direct agent-document and skill evidence

- Gloaguen et al., [Evaluating AGENTS.md](https://arxiv.org/abs/2602.11988).
- Wu et al., [SkillReducer](https://arxiv.org/abs/2603.29919).
- [Configuration Smells in Agent Context Files](https://arxiv.org/abs/2606.15828).
- [Agent READMEs: An Empirical Study of Context Files for Agentic Coding](https://arxiv.org/abs/2601.20404).

### Attention, format, and compression

- Liu et al., [Lost in the Middle](https://arxiv.org/abs/2307.03172).
- [IFScale](https://arxiv.org/abs/2507.11538).
- Sclar et al., [Prompt-format sensitivity](https://arxiv.org/abs/2310.11324).
- [Prompt Design at Scale](https://arxiv.org/abs/2607.19257).
- [Structured Context](https://arxiv.org/abs/2602.05447).
- [Notation Matters](https://arxiv.org/abs/2605.29676).
- Jiang et al., [LLMLingua](https://arxiv.org/abs/2310.05736),
  [LongLLMLingua](https://arxiv.org/abs/2310.06839), and
  [LLMLingua-2](https://arxiv.org/abs/2403.12968).
- [Prompt Compression in the Wild](https://arxiv.org/abs/2604.02985).
- [Telegraph English](https://arxiv.org/abs/2605.04426).
- [BabelTele](https://arxiv.org/abs/2606.19857).
- [Semantic Compression via Symbolic Metalanguages](https://arxiv.org/abs/2601.07354).
- [Context Codec Language](https://arxiv.org/abs/2605.17304).

### Adjacent compaction and representation work

- [Context Compaction Theory](https://arxiv.org/abs/2608.01326).
- [Rate-Distortion View of Memory Compaction](https://arxiv.org/abs/2607.08032).
- [Remember the Decision, Not the Description](https://arxiv.org/abs/2605.10870).
- [Self-Compacting Language Model Agents](https://arxiv.org/abs/2606.23525).
- [Parallel Context Compaction](https://arxiv.org/abs/2605.23296).
- [Training with Pseudo-Code for Instruction Following](https://arxiv.org/abs/2505.18011).
- [Code formatting and token efficiency](https://arxiv.org/abs/2508.13666).
- [DETAIL Matters](https://arxiv.org/abs/2512.02246).

### Current official platform guidance

- OpenAI, [Codex `AGENTS.md`](https://learn.chatgpt.com/docs/agent-configuration/agents-md),
  [Build skills](https://learn.chatgpt.com/docs/build-skills), and
  [current model guidance](https://developers.openai.com/api/docs/guides/latest-model).
- Anthropic, [Claude Code memory and instructions](https://code.claude.com/docs/en/memory)
  and [effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents).
- [Agent Skills specification](https://agentskills.io/specification).
- Cloudflare, [AI consumability](https://developers.cloudflare.com/style-guide/how-we-docs/ai-consumability/)
  and [Markdown for Agents](https://developers.cloudflare.com/fundamentals/reference/markdown-for-agents/).
