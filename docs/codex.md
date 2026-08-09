# Compressing LLM Documentation

**Design notes and research basis for a `compress-llm-documentation` skill**  
Research snapshot: **2026-08-09**

## Executive summary

The most effective way to compress `AGENTS.md`, `CLAUDE.md`, `SKILL.md`, memory files, and similar LLM-facing documentation is **not sentence shortening**. It is **context architecture**:

1. **Delete context that does not change agent behavior.**
2. **Keep only globally applicable invariants in always-loaded files.**
3. **Scope instructions by directory/file/task.**
4. **Use progressive disclosure:** tiny routing metadata → core workflow → conditional references/assets/scripts.
5. **Move deterministic procedures from prose into executable scripts/tests/configuration.**
6. **Compress semantically, not lexically:** preserve conditions, negation, precedence, literals, exceptions, defaults, and verification criteria.
7. **Prefer familiar Markdown/code over exotic compact notation unless benchmarks prove a net gain.**
8. **Measure task success + total trajectory cost, not just characters/tokens in the source file.**
9. **Use ablation:** if removing an instruction does not measurably hurt behavior, remove it.
10. **Treat compression as an optimization/evaluation loop, not a one-shot summarization task.**

A 2026 controlled study of repository context files is particularly important: LLM-generated context files often **reduced success rate while increasing execution cost by ~20–23%**, because they induced extra exploration, testing, and tool use. Human-written context sometimes improved success, but also increased steps/cost. The paper's practical conclusion is close to the right design goal for a compression skill: **repository context should contain only minimal requirements that the agent actually needs** [S10].

---

## 1. What is being optimized?

A normal document is optimized for a human reader:

- pedagogy;
- narrative continuity;
- redundancy for recall;
- motivation/rationale;
- friendly transitions;
- discoverability by browsing;
- visual presentation.

An LLM instruction/context file should instead optimize:

- **behavioral signal per token**;
- **retrieval/activation precision**;
- **low ambiguity**;
- **low contradiction rate**;
- **exact preservation of operational constraints**;
- **low context residency** for rarely needed information;
- **low downstream tool/trajectory cost**.

A useful objective is not simply:

```text
min tokens(document)
```

but:

```text
maximize Utility =
  task_success
  - λ1 * always_loaded_tokens
  - λ2 * conditional_tokens_loaded
  - λ3 * extra_tool_calls
  - λ4 * instruction_violations
  - λ5 * ambiguity/conflict
```

For a set of conditionally loaded files:

```text
ExpectedContextCost ≈ Σ P(load_i | workload) * tokens_i
```

This immediately suggests a non-trivial design rule:

> **Partition documentation by activation probability and co-access pattern, not merely by topic.**

A 2,000-token reference used in 2% of tasks is cheaper than a 150-token paragraph loaded in every task.

---

## 2. Core principle: behavioral necessity beats completeness

Before compressing a sentence, ask:

> **Would a competent target model likely behave incorrectly without this information?**

If **no**, delete it.  
If **unknown**, test by ablation.  
If **yes**, preserve it or make it easier to activate.

This matches the Agent Skills guidance: focus on project-specific conventions, domain procedures, non-obvious edge cases, and tool/API behavior the model would not know; avoid explaining general concepts the model already understands [S3].

### High-value information

Usually retain:

- non-obvious project invariants;
- exact build/test commands when nonstandard;
- dangerous operations and forbidden actions;
- repository-specific naming/path conventions;
- compatibility constraints;
- exact versions/thresholds when behavior depends on them;
- special fallback logic;
- exceptions;
- known model/agent failure modes;
- completion/verification criteria;
- tool choice when the default is non-obvious;
- hidden coupling between components;
- decision precedence.

### Low-value information

Usually delete or externalize:

- explanations of standard technologies;
- generic software-engineering advice;
- motivational prose;
- broad repository overviews discoverable with `ls`, search, LSP, or code indexing;
- duplicated README/API documentation;
- long lists of possible tools when one default is sufficient;
- repeated reminders of the same rule;
- obvious directory descriptions;
- examples that add no new behavior;
- history that does not affect current decisions;
- rationale that does not alter choices.

**Important:** “more context” is not automatically better. The 2026 AGENTS.md evaluation found that context files were followed, yet often caused agents to do *more unnecessary work* rather than solve tasks better [S10].

---

## 3. Context architecture: hot / warm / cold

Treat documentation like a cache hierarchy.

### Hot: always loaded

Examples:

- root `AGENTS.md`;
- root `CLAUDE.md`;
- skill `name` + `description`;
- first section/index of a memory file.

Keep only:

- universal invariants;
- top-level scope;
- critical gotchas whose trigger may not be recognizable in advance;
- routing instructions;
- exact default commands needed frequently;
- pointers to conditional material.

### Warm: task/path scoped

Examples:

- nested `AGENTS.md`;
- `.claude/rules/*.md`;
- `.github/instructions/*.instructions.md`;
- nested `.claude/skills/`;
- task-specific `SKILL.md`.

Load only when the agent enters a relevant directory, file type, or workflow.

### Cold: on-demand

Examples:

```text
references/
assets/
examples/
schemas/
runbooks/
archives/
```

Use explicit trigger-aware links:

```md
If API returns non-2xx, read `references/api-errors.md`.
For DB migrations, read `references/migrations.md` before editing.
```

Prefer this to:

```md
See `references/` for more information.
```

OpenAI, Anthropic, and the Agent Skills specification all increasingly rely on this **progressive disclosure** model [S2][S3][S5][S6].

---

## 4. `AGENTS.md` / `CLAUDE.md`: what should remain in the root?

A good root file should resemble an **interface contract**, not a handbook.

Recommended core:

```md
# Project instructions

## Commands
- Test: `...`
- Lint: `...`
- Build: `...`

## Invariants
- ...
- ...

## Scope
- `backend/**`: ...
- `frontend/**`: ...

## Gotchas
- ...
- ...

## Done
- Run ...
- Do not ...
```

Avoid auto-generated directory tours such as:

```md
src/ contains source code.
tests/ contains tests.
docs/ contains documentation.
```

unless there is a non-obvious exception.

### Why nested instructions matter

Codex can construct an instruction chain from global → repository → deeper directory guidance; closer files can override earlier guidance. Claude similarly supports hierarchical/path-scoped instructions [S1][S4].

Therefore, instead of:

```md
# root AGENTS.md
[400 lines of Java rules]
[300 lines of Vue rules]
[250 lines of Python rules]
```

prefer:

```text
AGENTS.md
backend/AGENTS.md
frontend/AGENTS.md
tools/AGENTS.md
```

or path-scoped rule files where supported.

### Keep conflicts close to zero

Conflicting instructions are particularly harmful. Current models still show substantial degradation on instruction-hierarchy conflict benchmarks. Compression should therefore include **cross-file contradiction detection**, not merely deduplication.

Normalize rules into comparable forms:

```text
scope | condition | modality | action | object | exception | source
```

Example:

```text
frontend/**/*.ts | before commit | MUST | run | pnpm lint | - | root/AGENTS.md
frontend/**/*.ts | before commit | MUST_NOT | run | pnpm lint | - | frontend/AGENTS.md
```

Flag as a semantic conflict even though the strings are different.

---

## 5. Skills: exploit progressive disclosure aggressively

Modern Agent Skills use a particularly compression-friendly architecture:

```text
skill/
├── SKILL.md
├── references/
├── scripts/
├── assets/
└── examples/
```

Only the small discovery metadata (`name`, `description`, sometimes path) is initially exposed; the full `SKILL.md` is loaded after activation, and deeper files only when needed [S2][S5][S6].

### Optimize the `description` separately

The description is not documentation; it is a **router feature vector**.

It should front-load:

- what the skill does;
- high-value trigger terms;
- boundaries / when not to use it.

Bad:

```yaml
description: A useful skill for working with documentation in many different situations.
```

Better:

```yaml
description: Compress and refactor LLM-facing Markdown instructions (`AGENTS.md`, `CLAUDE.md`, `SKILL.md`, memory/rules) while preserving operational constraints; use for token/context reduction, deduplication, progressive disclosure, and instruction-file restructuring.
```

OpenAI explicitly notes that skill descriptions may be shortened under catalog pressure, so the key use case and trigger words should appear early [S2].

### Recommended size policy

Agent Skills guidance currently recommends keeping `SKILL.md` roughly under **500 lines / 5,000 tokens**, with detailed material moved into references [S3].

For a compression skill, I would aim much lower:

```text
SKILL.md:       ~800–2,000 tokens
reference file: only when needed
scripts:        deterministic measurement/validation
examples:       only selected by case
```

This is a heuristic, not a platform limit.

---

## 6. Memory files are not policy files

Memory and instructions have different semantics.

Use instruction files for:

- required behavior;
- standards;
- policy;
- must/never constraints.

Use memory for:

- stable learned facts;
- recurring preferences;
- discovered commands;
- debugging findings;
- decisions and their current status.

OpenAI explicitly recommends keeping required team guidance in `AGENTS.md` or checked-in documentation rather than relying only on memory [S19].

Claude's memory documentation also treats memory/CLAUDE.md as context rather than hard enforcement, and recommends concise, specific files [S4].

### Compress memory as state, not narrative

Bad:

```md
Yesterday we tried X and then it failed because..., after that we discussed Y...
```

Better:

```md
## Current facts
- Build: `pnpm build`.
- Node 24 breaks plugin X; use Node 22.

## Decisions
- 2026-08-03: Keep REST v1 until mobile client migrates.

## Open
- Investigate cache invalidation in `foo.ts`.

## Superseded
- ~~Use Node 20~~ → Node 22 since 2026-07-18.
```

A memory compressor should explicitly remove:

- obsolete hypotheses;
- resolved debugging branches;
- repeated observations;
- stale plans;
- conversational chronology.

Preserve:

- current state;
- decisions;
- unresolved questions;
- evidence pointers;
- exact commands/identifiers.

---

## 7. Semantic compression before lexical compression

The safest compression pipeline is **two-level**.

### Level A — semantic/context compression

Remove or relocate whole information units:

- irrelevant sections;
- duplicates;
- general knowledge;
- discoverable facts;
- rarely used workflows;
- redundant examples;
- superseded rules.

This often yields the largest gains.

### Level B — lexical/syntactic compression

Only after Level A:

- shorten sentences;
- remove filler;
- convert prose to imperative rules;
- collapse repeated subjects;
- merge equivalent rules;
- use compact condition/action syntax;
- normalize terminology.

This ordering mirrors prompt-compression research: coarse-to-fine methods such as LLMLingua/LongLLMLingua allocate different compression budgets to different prompt components rather than deleting tokens uniformly [S11][S12].

---

## 8. Build a semantic IR before rewriting

A one-shot prompt such as “shorten this by 60% without losing meaning” is unsafe for operational documentation.

Instead, first extract a **semantic intermediate representation (IR)**.

Suggested record:

```yaml
id: R17
scope: frontend/**
type: invariant        # invariant|procedure|fact|gotcha|example|rationale|pointer
condition: modifying API clients
modality: MUST         # MUST|MUST_NOT|SHOULD|DEFAULT|FALLBACK|INFO
action: regenerate client
target: scripts/gen-api.sh
exceptions: none
literals:
  - scripts/gen-api.sh
priority: critical
source: AGENTS.md:44-48
```

Then regenerate compressed Markdown from the IR.

### Why this helps

It separates:

1. **meaning preservation**, from
2. **surface compression**.

It also enables deterministic validation:

```text
original IR == compressed IR
```

for high-priority constraints.

### Minimum semantic fields to preserve

Never accidentally drop:

- `MUST`, `NEVER`, `ONLY`, `BEFORE`, `AFTER`;
- conditions (`if`, `unless`, `when`);
- scope;
- exceptions;
- defaults;
- fallback behavior;
- precedence;
- numbers/limits;
- exact commands;
- paths;
- identifiers;
- versions;
- output schema;
- completion criteria.

These are **semantic anchors**.

---

## 9. Compression budgets by information class

Do not use one compression ratio for the whole document.

Suggested starting policy:

| Class | Compression tolerance | Treatment |
|---|---:|---|
| Exact command/path/version | ~0% | preserve verbatim |
| Safety/destructive constraint | 0–15% | wording may shrink, semantics exact |
| Project invariant | 10–30% | preserve condition + action |
| Gotcha | 10–40% | keep if non-obvious |
| Procedure | 20–50% | convert to steps/script |
| Example | 30–80% | retain only representative/edge examples |
| Rationale | 50–90% | keep only decision-changing rationale |
| Generic overview | 80–100% | usually delete |
| Tutorial/explanation | 70–100% | externalize unless required |
| Historical narrative | 80–100% | replace with current state/decision |

These percentages are heuristics for the skill, not empirical constants.

---

## 10. High-value rewrite operators

A `compress-llm-documentation` skill should implement explicit rewrite operators rather than “summarize”.

### R1 — delete throat-clearing

Before:

```md
In order to ensure that the codebase remains consistent and maintainable, it is important that developers make sure to run...
```

After:

```md
Before commit, run `pnpm lint`.
```

### R2 — condition → action

Before:

```md
When you are working on database migrations, you should first make sure that...
```

After:

```md
DB migration → run `python scripts/migrate.py --verify --backup` first.
```

Use arrows only if tokenizer/evals show no downside. Plain `If X: Y.` is safer across models.

### R3 — merge duplicated modality

Before:

```md
Never edit generated API files.
Do not modify generated API clients manually.
Generated API clients should not be edited.
```

After:

```md
Never edit generated API clients manually.
```

### R4 — default + exception

Before:

```md
There are several libraries that can be used. Usually we use pdfplumber. In scanned cases...
```

After:

```md
Default: `pdfplumber`; scanned PDF: `pdf2image` + OCR.
```

Agent Skills guidance similarly recommends providing a default rather than a menu of equal options [S3].

### R5 — replace prose with one executable example

A correct command or minimal code pattern can encode:

- tool;
- syntax;
- flags;
- ordering;
- naming;
- expected structure.

GitHub's analysis of >2,500 agent files similarly emphasizes exact commands and examples over vague explanations [S17].

### R6 — factor repeated conditions

Before:

```md
For frontend files, use pnpm.
For frontend tests, use pnpm.
For frontend builds, use pnpm.
```

After:

```md
`frontend/**`: use `pnpm` for install/test/build.
```

### R7 — canonicalize vocabulary

Choose one term:

```text
repository | repo | project source tree
```

and reuse it.

This improves both compression and semantic matching.

### R8 — move long templates to assets

Instead of describing a 30-line output format, use:

```md
For release notes, fill `assets/release-template.md`.
```

Short templates can stay inline; large conditional templates should be loaded on demand [S3].

### R9 — executable policy beats prose where possible

Instead of:

```md
Always ensure formatting is correct and imports are sorted...
```

prefer:

```md
Run `make verify`.
```

and let `make verify` encode formatter/linter/typecheck/test policy.

### R10 — preserve non-obvious gotchas in hot context

Do **not** move every exception into cold references.

If the agent cannot know that it should look for the exception until after it makes the mistake, keep the gotcha in `SKILL.md`/root instructions. Agent Skills explicitly highlights this case [S3].

---

## 11. Deterministic knowledge belongs in code/config, not prose

A major compression opportunity is **representation substitution**.

If a rule can be enforced or computed exactly, move it from natural language to:

- formatter configuration;
- linter rule;
- type system;
- schema;
- Makefile/task runner;
- script;
- pre-commit hook;
- CI;
- test;
- generated index.

Then the LLM documentation only needs the invocation/contract.

Example:

Before:

```md
Files must use LF endings. TypeScript uses two spaces. Imports must be sorted...
```

After:

```md
Run `pnpm lint:fix`; do not hand-format around formatter output.
```

Anthropic's Agent Skills design explicitly notes that deterministic operations can be cheaper and more reliable as code than as token generation [S6].

---

## 12. Do not confuse file-size compression with agent-cost compression

A document can become 30% shorter and still make the agent more expensive if it:

- becomes ambiguous;
- causes retries;
- triggers extra searches;
- introduces unfamiliar syntax;
- removes a crucial default;
- forces extra reference reads;
- causes parsing failures.

A 2026 benchmark of token-optimized structured formats found that TOON/TRON could reduce input tokens, but unfamiliar output formats sometimes reduced accuracy or caused parsing cascades that **increased total trajectory tokens** [S15].

Therefore evaluate:

```text
TotalCost =
  startup_context
  + loaded_references
  + tool_results
  + retry/reasoning cost
  + output tokens
```

not merely:

```text
tokens(compressed.md)
```

---

## 13. Exotic notation: use selectively

### Markdown

Default choice for instruction prose:

- familiar to coding models;
- strong training prior;
- supports headings, bullets, code, tables, links;
- easily diffed/grepped;
- works across most agent ecosystems.

### YAML / JSON

Good for:

- schemas;
- metadata;
- semantic IR;
- machine-generated configuration.

Bad for prose-heavy rules because repeated keys/quotes can add noise.

### TOON / TRON / compact structured formats

Potentially useful for **large repeated structured datasets**, not as a default replacement for Markdown instructions.

The 2026 “Notation Matters” benchmark found:

- input-side TOON/TRON could reduce total tokens;
- gains depended heavily on workload shape;
- unfamiliar formats sometimes reduced accuracy;
- output-side compression could trigger parse/retry cascades;
- no format was universally superior [S15].

Rule:

> **Use novel serialization only when repetition is high and target models are benchmarked on it.**

### Custom DSL

A custom DSL can win only if its definition cost is amortized.

Approximate decision:

```text
use DSL if:
  savings_per_occurrence * repetitions
  > DSL_definition_tokens + error/recovery_cost
```

For a small instruction file, custom shorthand usually loses.

### Pseudocode

Use when it compresses procedural logic more clearly than prose.

Good:

```text
for rule in rules:
  if rule.scope matches target:
    apply(rule)
```

Bad:

```text
∀r∈R: M(s(r),t)⇒A(r)
```

unless the notation is already conventional in the domain.

### Formulae

Good for:

- scoring;
- thresholds;
- deterministic relations;
- optimization objectives.

Do not translate ordinary procedural requirements into math merely to appear compact.

### Images

Usually a poor compression strategy for operational LLM documentation:

- not always loaded by the harness;
- cannot be grepped/diffed reliably;
- may incur multimodal token/processing cost;
- exact literals/commands are harder to recover.

Use an image only when **visual topology is the information** (architecture diagram, UI state, graph). Keep exact instructions in text.

---

## 14. English vs other languages

Do not assume that translating documentation into another language automatically saves tokens.

Tokenization efficiency varies substantially by tokenizer/model/language [S18]. English often has strong tokenizer/training support in programming ecosystems, but this is not universal.

Recommended policy:

```text
1. Default technical agent documentation to clear English when the project is multilingual.
2. Measure tokens using the actual target tokenizer(s).
3. Do not mix languages merely for token savings.
4. Preserve identifiers/commands verbatim.
5. Optimize for comprehension + behavior first, token count second.
```

A compression tool targeting multiple models should calculate:

```text
tokens_gpt
tokens_claude_estimate
tokens_qwen
tokens_gemini
```

where APIs/tokenizers are available, then optimize a weighted or worst-case score rather than one tokenizer only.

---

## 15. Position matters: front-load what must be noticed

Long-context research shows that models do not use all positions equally; relevant information in the middle of long contexts can be less reliably used (“lost in the middle”) [S13].

For always-loaded documentation:

### Put early

- purpose/scope;
- destructive-action constraints;
- default commands;
- critical invariants;
- routing/pointers.

### Put later

- detailed edge cases;
- conditional references;
- rationale;
- rare alternatives.

For skills specifically, the **description** should front-load trigger terms because discovery catalogs may truncate descriptions [S2].

Do not duplicate every critical instruction at both top and bottom; duplication costs tokens and can drift. Prefer a small “Critical” section near the beginning.

---

## 16. Tables: useful only when they encode a matrix

Tables are efficient when rows share the same schema.

Good:

```md
| Scope | Test | Lint |
|---|---|---|
| `ui/**` | `pnpm test` | `pnpm lint` |
| `api/**` | `pytest` | `ruff check .` |
```

Bad:

A one-row table or prose forced into many columns.

Rule:

> Use a table when it eliminates repeated labels across **multiple homogeneous rows**.

Otherwise bullets are usually easier to modify and less structurally brittle.

For large machine-generated tabular data, consider CSV/TOON-like formats only after token + comprehension benchmarks.

---

## 17. Links and file splitting

### Local links are “cold-storage pointers”

Good:

```md
OAuth failure → read `references/oauth.md`.
```

The pointer is cheap; the referenced file costs context only when opened in a progressive-disclosure workflow.

### But imports may not be lazy

Some harnesses support Markdown imports/references that are expanded immediately. Claude, for example, notes that imported instruction files can still enter context at launch [S4].

Therefore distinguish:

```text
link/pointer            -> potentially lazy
import/include          -> often eager
path-scoped instruction -> conditionally eager
```

A compression skill should understand the target harness before “splitting” a file.

### External URLs

Use external links for:

- upstream specs;
- citations;
- rarely needed current docs.

Avoid making critical operational behavior depend on network access. Prefer checked-in stable references for required rules.

---

## 18. Chunk by behavior, not arbitrary size

Naively splitting every 300 lines can make context worse.

Partition by **semantic affinity + co-access probability**.

Conceptually, build a graph:

```text
node = rule/section
edge_weight(a,b) = P(a and b are needed in same task)
node_weight = token_count
```

Then partition so that:

- frequently co-used nodes remain together;
- low-frequency clusters become separate references;
- always-needed rules stay in the core;
- cross-file duplication is minimized.

This is a better model than “one file per topic” because two different topics may always be needed together, while two subsections of the same topic may have very different activation rates.

---

## 19. A non-trivial technique: instruction ablation

Treat each instruction as a candidate feature.

For rule `r`:

```text
Δ(r) =
  Eval(with r)
  - Eval(without r)
```

Possible outcomes:

```text
Δ > threshold   -> retain
Δ ≈ 0           -> remove/externalize
Δ < 0           -> rule is harmful; rewrite/remove
```

Run ablations first on:

- generic style advice;
- repository overviews;
- broad “always inspect…” directives;
- duplicate test instructions;
- tool suggestions;
- long rationales.

This directly operationalizes the 2026 finding that extra context can induce unnecessary agent work [S10].

---

## 20. Another non-trivial technique: expected token residency

Classify every unit by how often it enters the model context.

Example:

| Unit | Tokens | Load probability | Expected tokens/task |
|---|---:|---:|---:|
| root rule | 80 | 1.00 | 80 |
| API reference | 1,500 | 0.05 | 75 |
| migration runbook | 900 | 0.02 | 18 |
| generic repo overview | 300 | 1.00 | 300 |

The 300-token overview is **4× more expensive** in expectation than a 1,500-token conditional reference.

This metric should drive restructuring.

---

## 21. Another non-trivial technique: semantic checksums

Before rewriting, extract a checksum of high-risk semantics:

```yaml
must:
  - run pnpm test before commit
never:
  - edit src/generated/**
numbers:
  - timeout=30s
  - node=22
paths:
  - scripts/gen-api.sh
precedence:
  - nested rule overrides root
fallbacks:
  - OCR only for scanned PDFs
```

After compression, re-extract and compare.

Fail the transformation if:

```text
missing literal
changed modality
changed number
lost exception
scope widened/narrowed unexpectedly
new contradiction
```

This is much safer than semantic-similarity scoring alone.

---

## 22. Another non-trivial technique: “discoverability tax”

Assign low retention value to facts that the agent can cheaply recover with tools.

Example:

```text
"The project uses React 19."
```

If `package.json` is one search away, this fact may not belong in always-loaded instructions.

But:

```text
"Do not upgrade React 19.1 → 19.2; internal plugin X breaks."
```

is non-discoverable operational knowledge and should remain.

Approximate retention score:

```text
value =
  behavioral_importance
  * non_discoverability
  * frequency
  / token_cost
```

This gives a principled basis for deleting repository overviews while preserving gotchas.

---

## 23. Another non-trivial technique: replace descriptive state with query instructions

Instead of snapshotting changing information:

```md
Current packages are A 1.2, B 4.5, C 7.1...
```

write:

```md
Read versions from `package.json`; never infer them from this file.
```

Advantages:

- shorter;
- no staleness;
- single source of truth;
- less contradiction.

For dynamic data, **teach the agent where to query**, not what the current answer is.

---

## 24. Another non-trivial technique: preserve evidence pointers, compress conclusions

When a rule depends on a long rationale:

Before:

```md
[300-word incident history explaining why generated code must not be edited]
```

After:

```md
Never edit `src/generated/**`; regenerate via `scripts/gen-api.sh`.
Rationale/history: `docs/incidents/generated-client.md`.
```

The operational rule stays hot; evidence becomes cold.

This pattern is especially useful for architecture decisions, security constraints, and compatibility workarounds.

---

## 25. Another non-trivial technique: temporal compaction for memory

Memory needs garbage collection.

Each fact gets:

```text
status: active | superseded | resolved | uncertain
last_verified: date
source: path/issue/commit
```

Compression pass:

```pseudo
for item in memory:
    if item.status in {superseded, resolved} and not needed_as_constraint:
        archive_or_delete(item)
    elif newer_item_supersedes(item):
        merge_into_newer(item)
    elif item.is_hypothesis and stale(item):
        downgrade_or_delete(item)
```

This is stronger than generic summarization because it explicitly models **state transitions**.

---

## 26. Another non-trivial technique: compact at semantic boundaries

For long-running agent histories, recent 2026 research suggests that fixed token-threshold compaction can fire at poor moments such as mid-derivation; adaptive compaction at subtask boundaries can preserve quality while reducing cost [S16].

The same idea applies to documentation refactoring:

Do not compress arbitrary character ranges.

Compress complete units:

- rule;
- procedure;
- decision;
- example;
- resolved incident;
- reference cluster.

---

## 27. Do not overcompress familiar syntax

There is a practical “compression cliff”.

Example:

Clear:

```md
If tests fail after dependency changes, run `pnpm install` once, then rerun tests.
```

Overcompressed:

```md
depΔ + test! => pnpm i; test↻
```

The second may save tokens but introduces:

- interpretation cost;
- tokenizer uncertainty;
- weaker training prior;
- harder maintenance;
- error risk.

**Rule:** compress natural language until each unit is close to a canonical instruction, but do not invent a private language unless its benefit is measured.

---

## 28. Avoid token folklore

Do not hard-code assumptions such as:

- `→` is always cheaper than `then`;
- YAML is always cheaper than JSON;
- Chinese is always more token-efficient than English;
- removing whitespace always helps;
- abbreviations always save tokens;
- one huge file is cheaper than references;
- a smaller input always yields a cheaper agent trajectory.

Tokenizers and model behavior differ.

The skill should **measure**, not guess.

---

## 29. Recommended `compress-llm-documentation` pipeline

```pseudo
INPUT:
  files
  target_harnesses
  target_models
  compression_mode = safe|balanced|aggressive
  eval_tasks? = optional

1. DISCOVER
   detect AGENTS/CLAUDE/SKILL/memory/rule semantics
   detect scope, inheritance, imports, references
   collect exact literals: commands, paths, versions, numbers

2. PARSE
   split into semantic units
   classify:
     invariant | procedure | gotcha | fact | rationale |
     example | template | overview | pointer | history

3. NORMALIZE
   canonicalize terms
   extract modality/condition/scope/action/exception
   build semantic IR
   detect duplicates/conflicts/superseded rules

4. SCORE
   for each unit estimate:
     behavioral_importance
     discoverability
     activation_probability
     fragility
     token_cost
     duplication
     staleness

5. ARCHITECT
   hot  -> root/core
   warm -> scoped/nested rule/skill
   cold -> references/assets/scripts/archive
   convert deterministic procedures to scripts/config where appropriate

6. COMPRESS
   apply class-specific budget
   use explicit rewrite operators
   preserve semantic anchors exactly

7. REORDER
   front-load scope/critical constraints/defaults/routing
   keep rare details conditional

8. VALIDATE
   re-extract semantic IR
   compare semantic checksum
   check paths/commands/versions/numbers
   detect new conflicts and broken links

9. MEASURE
   count tokens per target tokenizer where possible
   report:
     raw reduction
     always-loaded reduction
     expected-context reduction

10. EVALUATE
   run representative agent tasks if available
   compare:
     success
     cost
     tool calls
     retries
     instruction violations
     reference reads

11. ITERATE
   restore any rule whose removal harms behavior
   delete rules with zero/negative marginal value
```

---

## 30. Scoring model for a compression skill

A practical ranking function for each information unit:

```text
KeepScore =
  4 * Criticality
+ 3 * NonDiscoverability
+ 2 * ActivationFrequency
+ 2 * Fragility
+ 2 * ExceptionValue
- 2 * Duplication
- 2 * Staleness
- 1 * TokenCost
```

Then:

```text
high score   -> keep hot
medium       -> scope/load conditionally
low          -> delete/archive
```

Weights should be configurable.

A more rigorous implementation could learn weights from evaluation traces.

---

## 31. Suggested compression modes

### Safe

Use when documentation controls fragile production operations.

- delete obvious redundancy/general knowledge;
- deduplicate;
- shorten prose;
- preserve architecture;
- no aggressive information relocation without clear trigger.

Target heuristic: **20–40% token reduction**.

### Balanced

Default.

- semantic deletion;
- hot/warm/cold split;
- procedural compression;
- one representative example;
- move rationale/templates/references out of core.

Target heuristic: **40–65%**.

### Aggressive

Only with evals.

- heavy abstraction;
- strong examples pruning;
- semantic IR regeneration;
- optional compact structured formats for repeated data;
- ablation-driven deletion.

Target heuristic: **60–80%+**, but only if task performance remains acceptable.

The percentages are initial operating bands, not universal guarantees.

---

## 32. Validation metrics

A good skill should output more than “saved 48%”.

### Static metrics

```text
characters_before/after
bytes_before/after
lines_before/after
tokens_before/after
always_loaded_tokens_before/after
duplicate_rule_count
conflict_count
broken_reference_count
exact_literal_recall
semantic_anchor_recall
```

### Behavioral metrics

```text
task_success_rate
instruction_adherence
steps/tool_calls
tokens per solved task
wall time
retry count
reference reads
unnecessary exploration
tests invoked
files touched
```

### Skill-specific metrics

```text
trigger precision
trigger recall
false activation rate
description tokens
SKILL.md loaded tokens
conditional reference tokens
```

### Primary production metric

Prefer:

```text
cost_per_successful_task
```

over:

```text
tokens_per_prompt
```

---

## 33. Evals for the compression skill itself

Create a small benchmark corpus containing:

1. `AGENTS.md` with duplicated rules.
2. Nested instructions with precedence.
3. `CLAUDE.md` containing stale repo overview.
4. `SKILL.md` with bloated rationale.
5. Memory file with superseded facts.
6. Destructive-operation runbook.
7. Multiple exact versions/thresholds.
8. Conflicting root/nested rules.
9. Long output template.
10. Highly repetitive structured data.

For each example define a golden semantic set:

```yaml
required:
  - id: C1
    modality: NEVER
    action: edit
    target: src/generated/**
  - id: C2
    modality: MUST
    action: run
    target: scripts/gen-api.sh
```

Then evaluate the compressed result against this golden set.

---

## 34. Use differential agent testing

The most meaningful test:

```text
A = agent + original docs
B = same agent + compressed docs
C = same agent + no docs
```

Run the same representative tasks.

Compare:

```text
success(A,B,C)
cost(A,B,C)
steps(A,B,C)
violations(A,B,C)
```

This reveals three cases:

```text
B > A  -> compression improved signal/noise
B = A  -> cheaper equivalent
B < A  -> overcompression or bad restructuring
C >= A -> original docs may be unnecessary/harmful
```

This directly follows the experimental logic suggested by recent AGENTS.md research [S10].

---

## 35. Recommended skill directory

```text
compress-llm-documentation/
├── SKILL.md
├── references/
│   ├── preservation.md
│   ├── agent-files.md
│   ├── skills.md
│   ├── memory.md
│   ├── rewrite-operators.md
│   └── evaluation.md
├── scripts/
│   ├── count_tokens.py
│   ├── extract_constraints.py
│   ├── compare_constraints.py
│   ├── find_duplicates.py
│   ├── find_conflicts.py
│   └── check_links.py
├── assets/
│   ├── report-template.md
│   └── semantic-ir.schema.json
└── examples/
    ├── agents-before.md
    ├── agents-after.md
    ├── skill-before.md
    └── skill-after.md
```

### What belongs in `SKILL.md`

Only:

- trigger/scope;
- core objective;
- compression pipeline;
- preservation priorities;
- mode selection;
- required validation;
- conditional pointers.

Detailed theory and examples belong in references.

---

## 36. Candidate compact `SKILL.md` structure

```md
---
name: compress-llm-documentation
description: Compress/refactor LLM-facing Markdown instructions (`AGENTS.md`, `CLAUDE.md`, `SKILL.md`, memory/rules) while preserving operational semantics; use for token reduction, deduplication, scoping, progressive disclosure, and context-cost optimization.
---

# Compress LLM documentation

Goal: minimize expected LLM context/trajectory cost without changing required behavior.

## Preserve first
Never lose/change: scope; MUST/NEVER/ONLY; conditions; exceptions; precedence; defaults/fallbacks; exact commands, paths, identifiers, versions, numbers; output contracts; verification criteria.

## Process
1. Parse semantic units; extract constraints/literals.
2. Remove duplicates, generic knowledge, discoverable repo facts, stale/superseded content.
3. Classify: hot(always needed), warm(scoped), cold(on-demand).
4. Keep only universal invariants/gotchas/routing hot; move conditional detail to scoped files/references.
5. Prefer exact commands/examples/scripts over explanatory prose; convert deterministic policy to tooling where practical.
6. Rewrite as concise imperative/condition→action rules; provide one default, not menus.
7. Re-extract constraints and compare with source.
8. Measure target-model tokens and report reduction; if eval tasks exist, compare task success/cost/tool calls.

## Do not
- one-shot summarize fragile instructions;
- invent cryptic shorthand solely to save tokens;
- remove rationale that changes decisions;
- move a non-obvious gotcha behind a trigger the agent cannot recognize;
- assume split/import files are lazy-loaded;
- optimize characters instead of target tokenizer + task behavior.

Read `references/preservation.md` for fragile/safety-critical docs.
Read `references/agent-files.md` for AGENTS/CLAUDE/path-scope rules.
Read `references/skills.md` when compressing SKILL.md.
Read `references/memory.md` for persistent memory/history.
Read `references/evaluation.md` when behavioral evals are available.
```

This is intentionally a **core**, not the full final skill.

---

## 37. Recommended report from the skill

The compressor should explain *what type* of savings it achieved.

Example:

```md
## Compression report

- Tokens: 4,820 → 1,940 (-59.8%)
- Always-loaded: 4,820 → 910 (-81.1%)
- Conditional refs: 1,030
- Rules: 73 → 41
- Duplicates removed: 21
- Conflicts resolved/flagged: 3
- Exact literals preserved: 38/38
- Critical constraints preserved: 17/17

### Structural changes
- Moved Vue-only rules to `frontend/AGENTS.md`.
- Moved migration procedure to `references/migrations.md`.
- Replaced 19-line formatter policy with `make verify`.
- Removed discoverable repository tree overview.

### Risk
- 2 rationales heavily compressed.
- No behavioral eval suite was supplied; semantic validation only.
```

This makes the transformation auditable.

---

## 38. Anti-patterns the skill should detect

### A. Auto-generated repo encyclopedia

Symptoms:

- directory tree;
- dependency list copied from manifests;
- descriptions of obvious components;
- hundreds of framework best practices.

Action: delete most; keep non-obvious coupling/gotchas.

### B. Universal rule explosion

Symptoms:

```md
Always inspect all tests.
Always inspect all docs.
Always inspect all configs.
Always run full suite.
```

These directives can cause trajectory bloat.

Action: replace with task-conditioned rules.

### C. Repeated negative rules

Merge equivalent `don't`, `never`, `avoid`, `must not`.

### D. Tutorial inside an instruction file

Move to docs/reference; retain exact procedure.

### E. Multiple equal tool choices

Select default + fallback.

### F. Eager import disguised as modularization

Splitting a file is not compression if all pieces load at startup.

### G. Compression by cryptic abbreviations

Reject unless repeated enough and benchmarked.

### H. Generic AI instructions

Examples:

```md
Write clean code.
Follow best practices.
Be careful.
Think step by step.
```

Usually low-value for strong coding models unless the exact behavior has been empirically necessary.

### I. Stale mutable facts

Replace with query/source-of-truth instructions.

### J. Long examples with only one novel feature

Reduce to the smallest example demonstrating the behavior.

---

## 39. Priority order for compression

When asked to “make this file as small as possible”, use this order:

```text
1. remove harmful/unnecessary requirements
2. remove duplicate/superseded information
3. remove model-known/general knowledge
4. remove discoverable repository facts
5. scope conditionally relevant content
6. externalize large references/templates/examples
7. replace prose procedures with scripts/commands
8. merge equivalent rules
9. shorten sentences/labels
10. consider compact serialization only for large repeated structures
```

The last step is deliberately last. Architectural deletion/scoping is usually more valuable than punctuation tricks.

---

## 40. My strongest recommendations

### 1. Design for **minimum necessary context**, not maximum documentation

The best `AGENTS.md` is not the most informative one. It is the smallest set of information whose absence measurably causes bad behavior.

### 2. Optimize **expected loaded tokens**

Always-loaded 200 tokens can cost more than a 2,000-token reference used rarely.

### 3. Make the compressor **semantics-aware**

A generic summarizer is insufficient. Extract and revalidate modalities, conditions, scope, exceptions, literals, and precedence.

### 4. Add **ablation testing**

This is probably the highest-value differentiator from ordinary “prompt compression” tools.

### 5. Remove **discoverable context**

Repository maps and package facts are frequently cheap for an agent to query and expensive to preload forever.

### 6. Prefer **routing + retrieval** over monolithic memory

Hot index + scoped files + references scales better than a single compressed encyclopedia.

### 7. Move deterministic knowledge into **scripts/config/tests**

Do not spend reasoning tokens repeatedly reinterpreting rules that a tool can enforce exactly.

### 8. Treat format compression skeptically

New token-oriented formats can save tokens but also create parse/adherence failures. Familiar syntax has strong prior value.

### 9. Evaluate **cost per successful task**

Raw input-token reduction can be misleading when ambiguity increases retries/tool calls.

### 10. Compression must be reversible/auditable

Keep reports, semantic checksums, and optionally a mapping from compressed rule IDs to source locations.

---

## 41. Source-backed findings worth encoding into the skill

| Finding | Implication |
|---|---|
| Codex supports hierarchical `AGENTS.md` discovery and nested overrides [S1] | Scope rules locally rather than globally. |
| Skills use progressive disclosure [S2][S5][S6] | Keep routing metadata tiny; defer details. |
| Agent Skills recommends moderate detail and roughly `<500 lines / 5k tokens` for core SKILL.md [S3] | Treat exhaustive skills as a smell. |
| Claude recommends concise, structured instructions and path scoping [S4] | Split by applicability, not just readability. |
| LLM-generated repo context can hurt success and raise cost >20% [S10] | Delete non-essential context; use ablation. |
| Context quality degrades as irrelevant length grows [S7][S13] | More context is not neutral. |
| Prompt compression works best with differentiated budgets [S11][S12] | Preserve instructions/constraints more than examples/rationale. |
| Novel structured formats can save tokens but cause accuracy/parse regressions [S15] | Benchmark formats; never assume compression is free. |
| Semantic-boundary/adaptive compaction can outperform fixed thresholding [S16] | Compress complete semantic units, not arbitrary chunks. |
| Tokenization differs significantly across languages/models [S18] | Measure target tokenizers; avoid language folklore. |

---

## 42. Practical implementation roadmap

### v0 — deterministic/static

Implement:

- Markdown section parser;
- token counters;
- duplicate detection;
- exact literal extraction;
- link checker;
- simple rule patterns;
- before/after report.

### v1 — LLM semantic compressor

Add:

- semantic IR extraction;
- hot/warm/cold classification;
- rewrite operators;
- constraint re-extraction;
- semantic checksum validation.

### v2 — repository-aware

Add:

- detect nested instruction semantics;
- check whether facts are discoverable from manifests/config/code;
- propose path-scoped files;
- detect eager imports vs lazy references.

### v3 — behavioral optimizer

Add:

- task/eval harness;
- instruction ablation;
- differential A/B/C runs;
- cost-per-success optimization;
- learned retention weights.

### v4 — context graph optimizer

Add:

- co-access graph from agent traces;
- activation probability estimates;
- automatic file partitioning;
- expected-context-cost minimization.

This last stage is where the skill becomes genuinely more advanced than a normal Markdown summarizer.

---

## 43. References

**Official / primary guidance**

- **[S1] OpenAI — Custom instructions with AGENTS.md**  
  https://developers.openai.com/codex/guides/agents-md

- **[S2] OpenAI — Build skills / Agent Skills**  
  https://developers.openai.com/codex/skills

- **[S3] Agent Skills — Best practices for skill creators**  
  https://agentskills.io/skill-creation/best-practices

- **[S4] Anthropic — How Claude remembers your project (`CLAUDE.md`, memory, scoped rules)**  
  https://code.claude.com/docs/en/memory

- **[S5] Anthropic — Claude Code skills**  
  https://code.claude.com/docs/en/skills

- **[S6] Anthropic — Equipping agents for the real world with Agent Skills**  
  https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills

- **[S7] Anthropic — Effective context engineering for AI agents**  
  https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents

- **[S8] Anthropic — Writing effective tools for AI agents**  
  https://www.anthropic.com/engineering/writing-tools-for-agents

- **[S9] GitHub Docs — Effective custom instructions for Copilot code review**  
  https://docs.github.com/en/copilot/tutorials/customize-code-review

**Research / empirical evidence**

- **[S10] Gloaguen et al. (2026) — Evaluating AGENTS.md: Are Repository-Level Context Files Helpful for Coding Agents?**  
  https://arxiv.org/abs/2602.11988

- **[S11] Jiang et al. — LLMLingua: Compressing Prompts for Accelerated Inference of LLMs**  
  https://arxiv.org/abs/2310.05736

- **[S12] Jiang et al. — LongLLMLingua: Prompt Compression for Long Context**  
  https://arxiv.org/abs/2310.06839

- **[S13] Liu et al. — Lost in the Middle: How Language Models Use Long Contexts**  
  https://arxiv.org/abs/2307.03172

- **[S14] Pan et al. — LLMLingua-2: Efficient and Faithful Task-Agnostic Prompt Compression**  
  https://arxiv.org/abs/2403.12968

- **[S15] Notation Matters (2026) — Token-Optimized Formats in Agentic AI Systems**  
  https://arxiv.org/abs/2605.29676

- **[S16] Self-Compacting Language Model Agents (2026)**  
  https://arxiv.org/abs/2606.23525

- **[S17] GitHub — How to write a great agents.md: lessons from 2,500+ repositories**  
  https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/

- **[S18] Maksymenko et al. — Tokenization efficiency of current foundational LLMs**  
  https://pmc.ncbi.nlm.nih.gov/articles/PMC12380774/

- **[S19] OpenAI — Memories / customization guidance**  
  https://developers.openai.com/codex/customization/memories

**Additional useful research**

- Prompt Compression in the Wild (2026):  
  https://arxiv.org/abs/2604.02985

- Parallel Context Compaction for Long-Horizon LLM Agent Serving (2026):  
  https://arxiv.org/abs/2605.23296

- Agent Skills specification:  
  https://agentskills.io/specification

---

## 44. Bottom line

The useful mental model for `compress-llm-documentation` is:

```text
NOT:
  "shorten Markdown"

BUT:
  "compile human/legacy documentation into a minimal,
   scoped, behavior-preserving context program"
```

The optimal output is often **multiple smaller context layers + scripts/references**, not a shorter monolithic file.

A strong compressor should act more like a compiler:

```text
source docs
  -> semantic IR
  -> dead-context elimination
  -> deduplication
  -> scope analysis
  -> progressive-disclosure partitioning
  -> compact code generation
  -> semantic validation
  -> token/behavior eval
```

That architecture is substantially safer and more powerful than ordinary summarization, and it aligns with the direction taken by modern coding-agent harnesses, Agent Skills, and recent context-compression research.
