# Compressing LLM Documentation — Field Manual & Skill Blueprint

Research notes, rules, transformations and verification methods for compressing and rewriting
`AGENTS.md` / `CLAUDE.md`, `SKILL.md`, memory files, rule files and technical Markdown that is
read by machines rather than humans.

Intended use: the source material for a `compress-llm-documentation` skill.
Status: opinionated. Where I give a verdict on a technique, it is marked **Verdict:**.

---

## 0. TL;DR — the twelve rules that do 90% of the work

| # | Rule | Why |
|---|---|---|
| 1 | **Delete anything the model already knows.** | Zero surprisal = zero value, non-zero cost. |
| 2 | **Delete anything a linter, type checker, formatter or CI job already enforces.** | Deterministic tools do it free, at 100% consistency. Most common real-world smell (62% of files). |
| 3 | **Delete anything already stated in README/code/tests.** | Redundant context measurably *hurts*; unique context helps. |
| 4 | **Move, don't shrink.** Rarely-needed content → separate file, loaded on demand. Deterministic content → a script. | A script costs 0 context tokens and returns 3 lines. |
| 5 | **Replace payloads with pointers.** `src/auth/session.py::refresh_token` or `rg -n "class User" src/` instead of pasted code/schemas. | ~10 tokens, never stale. |
| 6 | **Never paraphrase identifiers.** Commands, paths, flags, env vars, error strings, versions survive verbatim. Compress only connective tissue. | Identifiers are the high-entropy payload. |
| 7 | **One rule per line, verb first, ≤ 15 words.** | Greppable, diffable, individually deletable, survives partial reads. |
| 8 | **Normalize modality to three levels: MUST / DEFAULT / OPTIONAL.** Delete everything below OPTIONAL. | Hedging ("consider maybe") burns tokens and transfers the priority decision to the model. |
| 9 | **Count imperatives, not words.** Budget ≤ ~40 always-on rules. | Instruction-following degrades with density; models drop rules silently. |
| 10 | **Close every enumeration and every procedure.** No "etc.", "and so on"; always a stop condition. | Open-ended language causes exploration, which is the main measured cost of context files. |
| 11 | **Order by violation frequency, not by logic.** Most-broken rule first; ≤ 5-line non-negotiables recap last. | Primacy + recency; the middle of a document is where attention sags. |
| 12 | **Compress from the original, never from the previous compression.** | Repeated LLM paraphrase is generational loss, like re-encoding a JPEG. |

Realistic outcome on a real, unmaintained `AGENTS.md`: **45–70% token reduction with no behavioural
regression**, most of it from rules 1–4 rather than from clever wording.

---

## 1. What the evidence actually says

This section matters because the popular advice ("generate a comprehensive context file with
`/init`") is contradicted by every controlled study published so far.

### 1.1 Context files are not free, and often not positive

ETH Zurich / LogicStar, *Evaluating AGENTS.md* (Feb 2026), 4 agents × 2 benchmarks
(SWE-bench Lite + a purpose-built AGENTbench of 138 issues from 12 repos that ship real
developer-written context files):

- LLM-generated context files **reduced** success rate by ~0.5–2 points on average and hurt in
  5 of 8 settings.
- Developer-written files improved success by ~4 points on average — a real but small gain.
- **Every** kind of context file increased the number of agent steps (+2.45 to +3.92) and
  inference cost (**+20–23%**).
- Reasoning-token usage rose 14–22% when a context file was present: following extra
  instructions makes the task *harder*.
- Context files did **not** speed up finding the right files. Time-to-first-touch of a
  file in the gold patch was unchanged. Overview sections do not work as overviews.
- The killer ablation: when the researchers deleted all other documentation from the repo,
  LLM-generated context files started *helping* (+2.7%). **The value of a context file is
  exactly the information that exists nowhere else.**
- Instructions *are* followed: a tool mentioned in the file was used ~1.6×/task versus
  <0.01×/task when unmentioned. So poor outcomes are not a compliance problem — they are an
  over-specification problem.

Practical reading: every sentence you write will be executed. Write fewer sentences.

Median developer-written file in that corpus: ~640 words, ~10 sections. Not 1,500 lines.

### 1.2 Six measurable configuration smells

UFMG, *Configuration Smells in AGENTS.md Files* (Jun 2026), 100 popular repos:

| Smell | Definition | Prevalence |
|---|---|---|
| **Lint Leakage** | Rules a linter/formatter already enforces (naming, indentation, import order, line length). | 62% |
| **Context Bloat** | File ≥ 200 lines; largest observed 1,477 lines. | 42% |
| **Skill Leakage** | Task-specific procedures kept always-on instead of in an on-demand file. Most leaked categories: testing, workflow, scaffolding. | 35% |
| **Conflicting Instructions** | Two rules that cannot both be satisfied (e.g. two different component directories). | 28 found, 16 confirmed |
| **Init Fossilization** | File generated once by `/init`, never edited again, in an actively developed repo. | 24% |
| **Blind Reference** | A path/link with no statement of what it contains or when to read it. | 16 found |

91 of 100 files had at least one smell. Smells co-occur: conflicting instructions + skill leakage
raise the probability of context bloat to ~83%.

**These six are the natural detector set for a compression skill.** They are cheap to check and
they map one-to-one onto fixes.

### 1.3 Attention is finite and non-uniform

- **Context rot** (Chroma, 2025; 18 frontier models): performance degrades non-uniformly as
  input grows, even on trivial retrieval and copying tasks. A 200K window can show real accuracy
  loss well before 50K tokens. Distractors and low lexical overlap between query and target make
  it worse. Long windows are not a licence to be verbose.
- **Lost in the middle** (Liu et al., TACL 2024): information at the beginning and end of a long
  input is used far more reliably than information in the middle.
- **IFScale** (Distyl AI, 2025), 20 models, 10→500 simultaneous instructions: best models reach
  only ~68% adherence at maximum density. A **primacy effect peaks around 150–200 instructions**;
  at extreme density, failures become uniform. Errors are overwhelmingly *omissions*, not
  mistakes — ratios above 30:1 for some models. Models silently drop rules; they do not announce it.
- **Format sensitivity** (Sclar et al., ICLR 2024; He et al., 2024): semantically identical prompts
  in different formats can swing accuracy by ~10 points on average and far more in the worst case;
  smaller models vary up to ~40% between plain text / Markdown / JSON / YAML. Larger models are
  more robust, but not immune. There is no universally optimal format, which is an argument for
  *consistency within a document* rather than for chasing an exotic encoding.

### 1.4 The synthesis

> A documentation file for an agent is not a reference. It is a **standing prompt** prepended to
> every task, whose every clause competes for a fixed attention budget and biases every subsequent
> token. Compression is therefore not a cost optimisation. It is a correctness optimisation.

---

## 2. Mental model

### 2.1 The value function of a token

```
value(chunk) = surprisal_to_model × task_relevance × cost_if_wrong
```

- `surprisal_to_model` → does the model already know this? (Python conventions: 0. Your bespoke
  migration ritual: high.)
- `task_relevance` → what fraction of sessions need it? (Deploy runbook in an always-on file: ~2%.)
- `cost_if_wrong` → is the failure recoverable? (Style nit: 0. Dropping a production table: ∞.)

If any factor is ~0, delete. If `task_relevance` is low but `cost_if_wrong` is high, **externalize
with a trigger** ("Before touching `db/migrations/`, read `docs/migrations.md`"). If
`cost_if_wrong` is high and relevance is high, it goes at the top, in capitals, and is repeated at
the bottom.

This is a per-claim analogue of what LLMLingua does per-token: use a model's own perplexity to find
which tokens carry no information, and drop them. We do the same at the level of *claims*, which is
both safer and automatable (§8.7).

### 2.2 Docs are priors, not references

Because instructions are followed (§1.1), documentation behaves like a set of soft constraints on
the policy, not like a book on a shelf. Consequences:

- Adding a rule you don't need costs you *twice*: tokens, plus the behaviour it induces.
- "Be thorough", "write clean code", "follow best practices" are not neutral filler — they measurably
  induce extra testing and exploration, which is exactly the +20% cost observed.
- Deleting a rule is a behavioural change and should be tested like one.

### 2.3 Two budgets, and the tighter one is not tokens

| Budget | Limit | Failure mode |
|---|---|---|
| Token budget | context window / cost | context rot, latency, price |
| **Instruction budget** | ~dozens before adherence decays | **silent omission** |

You can be under the token budget and over the instruction budget. A 120-line file containing 90
imperatives is worse than a 200-line file containing 25 imperatives and 5 examples.

**Measure: `rg -c '^\s*[-*]?\s*(MUST|NEVER|Always|Never|Do not|Don.t|Use|Run|Avoid|Prefer|Ensure)'`**

### 2.4 Lossy compression: the invariant set

Compression is lossy by definition. Define upfront what must be **bit-exact** and what may be
paraphrased freely.

**Never touched (extract into an inventory before you start rewriting):**

- shell commands, flags, and their exact spelling
- file paths, directory names, glob patterns
- symbol names, env vars, config keys, ports
- version numbers and pins
- error strings and log lines used for matching
- external URLs
- explicit prohibitions ("never force-push to main")
- numeric thresholds, timeouts, limits

**Freely compressible:** motivation, history, restated general knowledge, tutorials, apologetics,
transition sentences, repeated preambles, "as mentioned above", section intros, praise for the
codebase.

A compression pass that changes an identifier is a **bug**, not a stylistic choice. The skill
should diff the identifier inventory before/after and fail on any delta.

---

## 3. LLM-optimized vs human-optimized documents

The two audiences want opposite things more often than people expect.

| Dimension | Human-optimized | LLM-optimized |
|---|---|---|
| **Onboarding** | Explains domain, motivates, builds intuition | Assumes a senior contractor who has read 10M repos but not yours; only the *delta* matters |
| **Redundancy** | Aids memory, forgiveness for skimming | Dilutes attention; creates contradiction risk; **except** for the top-3 rules (deliberate top+bottom repetition) |
| **Ordering** | Narrative / conceptual | Primacy & recency; most-violated first, non-negotiables recapped last, reference in the middle |
| **Ambiguity** | Resolved by asking a colleague | Resolved by *invention* — the model will fill the gap with a plausible policy and act on it |
| **Hedging** | Politeness, humility | "Should / may / consider" reads as optional. Promote to MUST or delete |
| **Examples** | One illustrative example | 2–3 **diverse boundary** examples beat any amount of description; the model interpolates the middle |
| **Rationale** | Always welcome | Keep only when it lets the model derive an *unwritten* rule; delete pure justification |
| **Navigation** | ToC, search, skimming | Explicit "read `X` when `Y`"; a bare path is ignored or eagerly slurped |
| **Prose vs table** | Prose reads better | Tables/decision matrices declare field names once and remove ambiguity |
| **Length** | Longer = more thorough | Longer = worse, monotonically, past a small threshold |
| **Decoration** | Emoji, banners, ASCII art, badges | Pure token cost; box-drawing characters are expensive and carry no meaning |
| **Freshness** | Stale sections are mildly annoying | Stale sections are *actively executed*, causing real damage |

**Corollary:** stop trying to write one file for both audiences. `README.md` is for humans;
`AGENTS.md` is for agents; they should share almost no text. Cross-linking beats merging.

---

## 4. Layer 1 — Architecture (compress by moving)

Structural compression beats textual compression by an order of magnitude. Do this first; only then
start rewriting sentences.

### 4.1 The four tiers

| Tier | What | Cost | Budget |
|---|---|---|---|
| **T0 — Always resident** | Skill `name` + `description`; the root rules file | Paid every single turn | `description` ≤ 1024 chars; root file **≤ 150 lines / ~1.5–2K tokens** |
| **T1 — Loaded on trigger** | `SKILL.md` body, scoped `AGENTS.md` in subdirectories | Paid once per relevant session | ≤ 500 lines |
| **T2 — Read just-in-time** | `reference/*.md`, schemas, runbooks, API dumps | Paid only if actually needed | Any size; add a ToC above 100 lines |
| **T3 — Executed, never read** | `scripts/*.py`, validators, generators, migrations | **0 tokens**; only the output costs | Any size |

> **The single largest compression move available to you is promoting content from T1 to T3.**
> A 400-line validation script that prints `OK` or 3 error lines has an effective compression ratio
> of roughly 100:1 versus explaining the same rules in prose — and it is deterministic.

### 4.2 Progressive disclosure rules

- **One level deep.** References must link directly from the entry file. Nested chains
  (`SKILL.md → advanced.md → details.md`) get partially read — agents preview with `head -100`
  and silently work from incomplete information.
- **Pitch every link.** `path — what's inside — when to read it`. This is the fix for Blind
  Reference. A path with no pitch is either ignored or loaded unnecessarily; both are failures.
  - Bad: `See docs/plugin-reorg.md for details.`
  - Good: `docs/plugins.md — plugin lifecycle + hook signatures. Read before adding or renaming a hook.`
- **Split by domain, not by size.** `reference/finance.md`, `reference/sales.md` — so a question
  about revenue never loads marketing schemas.
- **Add a grep recipe** for large reference files: `rg -i "revenue" reference/finance.md`. Cheaper
  than a full read and teaches the agent the retrieval pattern.
- **ToC above 100 lines**, so a partial read still reveals the file's full scope.
- **Descriptive filenames.** `form_validation_rules.md`, never `doc2.md`. The filename is part of
  the retrieval index.
- **Forward slashes always**, even for Windows projects.

### 4.3 Delegation: the compression hierarchy

For any rule, ask "what is the cheapest enforcement mechanism?" and use the highest one that works:

```
1. Make it impossible          → types, schema, API design, file permissions   (0 tokens, 0 failures)
2. Make it automatic           → formatter, codegen, pre-commit hook            (0 tokens)
3. Make it checkable           → linter, test, CI gate, validator script        (0 tokens, ~5 tokens of output)
4. Make it a runnable command  → "run scripts/check_x.py"                       (~10 tokens)
5. Make it a written rule      → one line in AGENTS.md                          (~15-40 tokens, forever)
6. Make it prose               → paragraph                                      (avoid)
```

Every rule sitting at level 5 or 6 that could live at 1–3 is pure waste. This one table is
responsible for most of the deletions in a typical file.

### 4.4 Pointer over payload

| Instead of | Write | Saving |
|---|---|---|
| 40-line pasted function | `src/auth/session.py::refresh_token` | ~95% |
| Full DB schema dump | `rg -n "CREATE TABLE" migrations/ \| head -40` | ~98% |
| Enumerated directory tree | 3 entry points + `rg --files -g '*.tsx' src/` | ~90% |
| Copied API docs | `reference/api.md — full endpoint list; read when adding a route` | ~95% |
| Config file contents | `config/app.yaml (env overrides win; see \`Settings\` in src/config.py)` | ~90% |

Prefer **symbol references or grep patterns over line numbers** — line numbers rot on the next
commit, symbol names usually don't.

### 4.5 Scoping

Nested rule files are conditional loading for free: the closest file to the edited file wins in
most harnesses. A rule that only applies to `frontend/` belongs in `frontend/AGENTS.md`, not in the
root file that every backend task also pays for.

Root file = organization-wide invariants only. Everything else pushes down.

### 4.6 Cache-aware layout (a genuinely non-obvious win)

Prompt caching keys on an exact prefix. Therefore:

- Put **stable** content (invariants, commands, architecture constraints) **at the top**.
- Put **volatile** content (current sprint focus, temporary workarounds, WIP notes) **at the bottom**.
- Editing line 3 invalidates the cache for the whole file; editing the last line invalidates almost
  nothing.
- Keep a shared preamble **byte-identical** across repos if you reuse one — whitespace differences
  destroy cache hits.
- This also argues against reordering churn: a compression skill should be **idempotent** (§9.4) so
  it doesn't re-shuffle a file on every run and nuke the cache.

---

## 5. Layer 2 — Content (what to delete)

### 5.1 Classify every block before touching it

Run this as an explicit pass; it makes the deletions defensible.

| Class | Definition | Default action |
|---|---|---|
| **INVARIANT** | Always true, always relevant, expensive to violate | Keep, hoist to top |
| **COMMAND** | Exact command / path / flag | Keep verbatim |
| **RULE** | Project-specific constraint | Keep if not tool-enforceable |
| **PROCEDURE** | Ordered multi-step workflow | Keep if hot path; else → T2 file |
| **EXAMPLE** | Input/output pair or snippet | Keep 2–3 diverse; delete typical/redundant ones |
| **RATIONALE** | Why a rule exists | Keep **only if generalizing**; else delete |
| **REFERENCE** | Schemas, tables, enumerations | → T2 file with a pitch |
| **AMBIENT** | General knowledge (what a PDF is, how git works) | Delete |
| **REDUNDANT** | Already in README / code / tests / another rule file | Delete, link instead |
| **TOOL-ENFORCED** | Handled by linter/formatter/CI/types | Delete |
| **DECORATIVE** | Badges, banners, emoji headers, ASCII art, praise | Delete |
| **STALE** | Refers to code/tooling that no longer exists | Delete (verify first) |
| **CONFLICTING** | Contradicts another rule | Escalate to human — never silently pick one |

### 5.2 The delete-on-sight list

- "Follow best practices", "write clean, maintainable code", "use descriptive names", "handle errors
  properly", "be thorough", "think step by step", "make it production-ready".
- Explanations of standard tools, languages, or file formats.
- A "Project Overview" that enumerates directories. Empirically it does **not** help agents find
  files faster. Replace with ≤ 2 lines of orientation plus the 3 real entry points.
- Style/formatting rules (indentation, quotes, import order, line length, camelCase vs snake_case).
- Restated package.json / pyproject scripts.
- Anything phrased "you may want to consider possibly…".
- Dated statements ("before August 2025, use the old API"). Replace with a
  `<details><summary>Legacy</summary>` block, or delete.
- Duplicate rules stated in three places with three different wordings — hoist to one.
- Multi-option menus ("you could use pypdf, or pdfplumber, or PyMuPDF, or…"). Give one default plus
  a single named escape hatch.

### 5.3 Rationale: the sharp rule

Keep a *why* **iff** it lets the model derive a rule you didn't write.

- Delete: "We use `uv` because it's faster." → adds nothing; the rule alone suffices.
- Keep: "Migrations are append-only because prod replays them from zero on restore." → the model can
  now correctly infer to never edit an applied migration, never reorder, and to add a new one for a
  fix — three unwritten rules for ~14 tokens.

Rationale that generalizes is the **highest** compression ratio content in the document. Rationale
that merely justifies is the lowest.

---

## 6. Layer 3 — Rewriting transformations

Twenty transformations, roughly in order of ROI. Each is mechanical enough to be a step in a skill.

### T1. Verb-first, one rule per line

```diff
- Before pushing your changes, it is generally recommended that you make sure
- the type checker has been run, since CI will otherwise fail.
+ Run `pnpm typecheck` before pushing. CI fails otherwise.
```
21 → 9 words. Also: greppable, diffable, individually deletable.

### T2. Modality normalization

Map every modal to a three-value enum and delete the tail.

| Found | Becomes |
|---|---|
| must / never / always / required / critical | **MUST** / **NEVER** |
| should / prefer / recommended / typically | *default* (plain imperative) |
| may / can / optionally / feel free | OPTIONAL — usually delete |
| consider / it might be nice / you may want to | **delete** |

Rule of thumb: if you can't decide whether it's MUST or delete, it's delete.

### T3. Prose → decision table

```diff
- If the change touches the database, you should run the migration verifier and
- then the full test suite. For API changes, regenerate the OpenAPI spec and run
- contract tests. Otherwise, unit tests are enough.
+ | Change touches      | Required before commit                          |
+ |---------------------|-------------------------------------------------|
+ | `db/migrations/`    | `scripts/migrate.py --verify` + full suite       |
+ | `api/`              | `make openapi` + `pytest tests/contract`         |
+ | anything else       | `pytest tests/unit`                              |
```
Removes ambiguity about overlap and precedence, which prose cannot express cheaply.

### T4. Prose → guarded pseudocode

For branching policies with priority, a guard ladder beats a table:

```
on task:
  if touches(secrets/ | .env)          -> STOP, ask human
  elif touches(db/migrations/)         -> run scripts/migrate.py --verify
  elif touches(api/openapi.yaml)       -> make openapi && pytest tests/contract
  else                                 -> pytest tests/unit -q
  always: never commit to main; open a PR
```
**Verdict: strongly recommended.** Pseudocode is the densest unambiguous encoding of conditional
policy, models parse it natively, and it forces you to state precedence and defaults.

### T5. Prose → grammar (for output formats)

```
commit := <type>(<scope>): <subject>
type   := feat | fix | chore | docs | refactor | test
subject:= imperative, lowercase, ≤ 72 chars, no trailing period
```
Plus 2–3 real examples. Replaces half a page of description, and is checkable by a hook.

### T6. Prose → type signatures

Types are pre-compressed specifications. For any API surface:

```python
def render(report: Report, *, fmt: Literal["md","html"] = "md", charts: bool = True) -> Path
```
carries argument names, types, optionality, defaults, keyword-only-ness and return type in one line.
Even for dynamically typed or untyped code, write a stub-style signature block as a documentation
device.

### T7. Prose → formula

Quantitative rules compress brutally well:

```
retry: delay = min(2^n · 100ms, 30s), n ≤ 5, jitter ±20%
cache TTL = 300s (hot) | 86400s (static assets)
page size: 50 default, 200 max
```
**Verdict: recommended.** Formulas remove the "roughly / approximately" hedging that prose
invites, and they are unambiguous under partial reads.

### T8. Repeated bullets → table

If you have N items × M attributes, a table declares the attribute names once. This is the same
insight behind token-efficient serialization formats (see §8.4): schema-once, rows-after.

### T9. Hoist repeated preconditions

Four sections each beginning "make sure the venv is active" → one **Invariants** block at the top.
Factoring, exactly as in code.

### T10. Close enumerations

```diff
- Run the linters, formatters, etc.
+ Run `ruff check .` and `ruff format --check .`
```
"etc." is an instruction to the model to invent. Inventing is exploration; exploration is the
measured cost.

### T11. Add stop conditions

Every procedure gets a termination criterion and a failure branch.

```
3. Re-run `scripts/validate.py`. Stop when it prints OK.
   If it fails 3× on the same error, stop and report — do not attempt a workaround.
```
High value per token: directly counteracts the over-exploration that context files induce.

### T12. Negative → positive replacement

Models handle "do X" more reliably than "don't do Y", and distractor/negative requirements are a
known weak spot.

```diff
- Don't use `requests` for HTTP calls.
+ Use `httpx` for all HTTP calls.
```
Reserve **NEVER** for genuine cliffs (data loss, security, irreversibility) and keep that list under
about 7 items so it stays salient.

### T13. Delete the section intro

"This section describes how we handle testing." → delete. The heading already said that.

### T14. Collapse tutorial into command

```diff
- First install the package with pip, then import the reader class, then open
- the file, and finally iterate over the pages to extract text.
+ ```python
+ import pdfplumber
+ with pdfplumber.open("f.pdf") as pdf: text = pdf.pages[0].extract_text()
+ ```
```
~150 tokens → ~50, and it is now copy-pasteable.

### T15. Example curation: diversity over quantity

Keep the canonical case, one boundary case, one failure case. Drop near-duplicates. Prefer a real
excerpt referenced by path over a synthetic one pasted inline.

### T16. Consistent terminology

Pick one term and never vary it: always "endpoint", never a mix of "endpoint / route / URL / path".
Synonym drift forces the model to resolve coreference and creates false distinctions. It also
breaks grep, which is how agents actually navigate.

### T17. Headings as retrieval keys

Headings are the index. Use the phrase an agent would search for.

```diff
- ## Verification and Quality Assurance Considerations
+ ## Running tests
```
Keep to H2/H3. Make headings self-contained — an agent may see only that slice.

### T18. Frontmatter for machine metadata

Move "this applies when…" out of prose and into structured frontmatter: `name`, `description`
(what + **when**, third person), `scope`/`globs`, `version`, `last-verified`.

The `description` is the only part of a skill that is always resident — it deserves more editing
effort per character than anything else in the file. It must contain both the capability and the
trigger vocabulary, in third person ("Extracts …. Use when the user mentions PDFs, forms, …"),
because it is what selection is performed against.

### T19. Deliberate redundancy for the top three

Against the general no-repetition rule: repeat your three most safety-critical rules in a short
closing block. Primacy covers the top, recency covers the bottom, and the middle is where rules go
to die. Cost: ~30 tokens. Worth it.

### T20. Legacy folding

```markdown
<details><summary>Legacy v1 API (removed 2025-08)</summary>
`api.example.com/v1/messages` — no longer supported.
</details>
```
Keeps the historical answer available to a human without spending prime attention on it. Better
than a dated conditional, which becomes wrong rather than merely old.

---

## 7. Layer 4 — Token-level micro-optimization

**Honest framing: this is the least valuable layer.** It typically yields 3–8% after layers 1–3
have yielded 50%. Do it last, never at the cost of clarity, and never on identifiers.

| Technique | Effect | Note |
|---|---|---|
| Common words over rare ones ("use" not "utilize") | small | rare words fragment into 3–5 tokens |
| Digits not words ("3" not "three") | small | |
| Avoid long ALL-CAPS spans | small | caps tokenize worse; reserve caps for MUST/NEVER keywords where the emphasis buys compliance |
| ASCII punctuation only | small | smart quotes, em dashes, non-breaking spaces cost extra tokens and break diffs |
| ≤ 2 levels of bullet nesting | small | each indent level is tokens on every line |
| No manual line wrapping | small | hard wraps insert newline tokens and break grep |
| Delete trailing whitespace, collapse blank runs | tiny | |
| Drop badges, banners, emoji headers, box-drawing art | **medium** | decorative Unicode is surprisingly expensive |
| Consistent identifier casing | small + cache | improves prefix-cache hits |
| Tables instead of repeated key names | **medium** | see T8 |

Estimation heuristic when you can't count: **English Markdown ≈ 3.5–4 chars/token**; code ≈ 3
chars/token. For anything real, count properly (`tiktoken`, or the provider's token-count endpoint).

**Anti-tip:** do not compress by removing spaces after punctuation, abbreviating words, or dropping
articles wholesale. Telegraphic English ("run test suite ensure pass before push") saves ~10% and
measurably increases misparses. The savings are not worth it.

---

## 8. Non-trivial techniques, with verdicts

This section exists because most of these ideas are *tempting* and several are traps.

### 8.1 Writing docs in a more token-dense natural language

Chinese, Japanese and Korean encode more meaning per token than English for prose.

**Verdict: don't.** Instruction-following is best-attested in English; your identifiers, error
strings and code are English anyway, so you create a two-language document; grep suffers; and
diffs become unreviewable for part of the team. The 20–30% token saving is not worth a measurable
adherence risk. Exception: if the model must *produce* output in language X, your few-shot examples
should be in language X.

### 8.2 A private symbol legend (emoji / arrows / custom notation)

`⛔` = never, `⚡` = performance-critical, `→` = then.

**Verdict: partially.** Arrows (`→`), comparison operators (`≤`, `≥`) and `|` for alternation are
cheap, universally understood, and genuinely compress. **Custom emoji legends are a net loss**: the
legend itself costs tokens, emoji are multi-token, and you add a decode step to every read. If you
find yourself writing "Legend: 🔴 = ..., 🟡 = ...", you have invented a worse table.

### 8.3 Diagrams, images and Mermaid

- A **linked image** costs ~0 tokens if never opened and 800–1600+ if opened; most agents won't
  open it; and it cannot be grepped or diffed.
- **Inline Mermaid/ASCII diagrams** cost real tokens and usually lose to a 4-line edge list.

```diff
- [30 lines of mermaid flowchart]
+ ingest → validate → normalize → persist → index
+ validate fails → quarantine/ (manual review; never auto-retry)
```

**Verdict:** for machine-read docs, prefer a **text-structured graph** (edge list, arrow chain,
adjacency table). Keep the rendered diagram in the human README, or as a linked `.mmd` file (T2,
zero cost until needed). **Exception:** genuinely visual tasks — UI layout, PDF form geometry,
chart styling — where rendering to an image and letting the model *look* is the right move.

### 8.4 Exotic serialization formats (TOON, CSV, compact JSON)

For bulk uniform data, schema-once formats really do cut 30–60% versus JSON. But independent
benchmarks are mixed: on nested/non-uniform data, ultra-compact formats have scored *below* JSON,
YAML and Markdown for comprehension, and in one evaluation the **most accurate** format was also
one of the most verbose (Markdown key-value). There is a compression/comprehension frontier and it
is possible to fall off it.

**Verdict:** for *documentation*, stay in Markdown. Use a **Markdown table** for uniform records —
it sits at a good accuracy-per-token point and needs no format instructions ("prompt tax"). Reach
for CSV/TOON only for large uniform payloads passed as data, not for rules an agent must obey.

### 8.5 Reverse Chain-of-Density

Chain-of-Density (Adams et al., 2023) iteratively adds entities to a summary at fixed length.
Invert it for compression: **fix the information, shrink the length.**

```
loop until a pass fails:
  1. list every DIRECTIVE and IDENTIFIER in the current draft   (the invariant set)
  2. rewrite the draft ~20% shorter, preserving that set exactly
  3. re-extract the set; if anything is missing or altered -> revert, stop
```
Practically, 3–4 passes converge; the 5th usually starts dropping directives, which is your signal.

### 8.6 The amnesia probe (empirical redundancy detection)

The most powerful and most automatable technique in this document.

```
for each claim C in the doc:
    ask a fresh model (no doc): "In a repo like this, how would you do <task implied by C>?"
    if the answer already matches C  -> C is AMBIENT       -> delete
    if the answer contradicts C      -> C is HIGH VALUE    -> keep, and put it near the top
    if the model asks / guesses      -> C is USEFUL        -> keep, compress
```
This uses the model as the compressor's dictionary — the same principle as perplexity-based prompt
compression, lifted to the claim level. Use a *cheap, fast* model for the probe: what a small model
already knows, a large one certainly does.

Extension: run the probe *with the codebase available*. If the model finds the answer by reading
the repo in one or two tool calls, the doc line is redundant with the code and should become a
pointer (§4.4), not a payload.

### 8.7 Budget allocation by section (borrowed from LLMLingua)

Prompt compressors allocate different compression ratios to different prompt parts. Do the same:

| Section type | Target compression | Rationale |
|---|---|---|
| Commands, paths, invariants | **0%** | lossless-only zone |
| Prohibitions / safety rules | 0–10% | wording carries the force |
| Procedures | 20–40% | keep steps, cut narration |
| Examples | 40–70% | keep 2–3 diverse, drop the rest |
| Reference tables | 20–40% | or relocate whole to T2 |
| Rationale | 60–90% | keep only generalizing why |
| Overview / motivation | 90–100% | usually deletable outright |

Publishing these ratios inside the skill makes its behaviour predictable and reviewable.

### 8.8 Generational loss (the JPEG rule)

Every LLM paraphrase pass drifts semantics slightly. Compressing an already-compressed file
repeatedly produces the documentation equivalent of a re-encoded JPEG: fluent, shorter, subtly
wrong. **Always compress from the newest human-authored source**, keep the original in git, and
make the operation idempotent (§9.4) so a second run is a no-op rather than a second generation.

### 8.9 Docs-as-tests (the real fix for staleness)

Init Fossilization is not a writing problem, it is a lifecycle problem. Make doc claims falsifiable:

- Every command in the file is extracted by a script and executed in CI (`--help` at minimum).
- Every path referenced must exist (`test -e`).
- Every `path::symbol` reference must resolve (ctags / LSP / `rg`).
- Optional: tag each section with the commit SHA it was last verified against, not a date.

A documentation line that CI can verify never rots. This converts documentation maintenance from
discipline into a build failure.

### 8.10 The removal ledger (anti-ratchet)

After compressing, write `AGENTS.notes.md` — **not loaded by any agent** — listing what was removed
and why ("style rules → enforced by ruff.toml"; "overview → measured as non-helpful"). Without it,
the next person re-adds everything and the file re-inflates within two quarters. Cost: zero context
tokens.

### 8.11 Conflict graph

Extract rules as `(scope, subject, directive)` triples and check for contradictions on the same
`(scope, subject)`. Automated detection of conflicting instructions runs at modest precision
(~57% in published work), so: **surface conflicts to a human, never auto-resolve.** Silently
picking one is how you get a doc that says the opposite of what the team decided.

### 8.12 Sort by violation frequency

If you have agent transcripts, mine them: which rules were broken, how often, at what cost. Sort the
rule list by `violations × cost`. Logical grouping is a human convenience; attention position is a
machine constraint. This is also the only honest way to decide what earns a MUST.

---

## 9. Verification — proving you didn't break it

Compression without verification is vandalism. Four tests, in increasing cost.

### 9.1 Identifier diff (cheap, mandatory, automatable)

Extract the invariant set (§2.4) from before and after; the sets must be equal. Any deletion must be
explicitly approved; any *addition* is a hallucination and fails the run.

```
before_ids - after_ids  ->  must be ∅ or explicitly approved
after_ids  - before_ids ->  must be ∅   (compressor may not invent)
```

### 9.2 Fact-recall test

Generate 15–30 questions from the **original** ("What runs the tests?", "Where do migrations live?",
"What must never be committed?"). Answer them using **only the compressed file**. Must-Keep facts
require 100%; Nice-to-Have facts a stated threshold. This catches lossy deletion directly.

### 9.3 Behavioural A/B

Run 3–5 representative tasks twice — once with the original file, once with the compressed one — and
diff the traces:

- same tool sequence (allowing for reordering)?
- same files touched?
- same final artifact / tests passing?
- **fewer** steps and tokens? (this is the actual goal, per §1.1)

Equal or better outcome at lower cost = ship. This is also the only test that can show the
compression *improved* behaviour, which it frequently does.

### 9.4 Idempotency and convergence

Run the skill twice. The second run must produce a byte-identical file. A non-idempotent compressor
drifts semantics (§8.8), thrashes prompt caches (§4.6) and produces unreviewable diffs. Make this a
hard test in the skill's own eval suite.

### 9.5 Rule-enumeration probe (catches silent omission)

Give a fresh model the compressed file and ask it to list every rule it will follow. Compare with
the intended rule set. This detects the IFScale failure mode — rules present in the text but
dropped under load — and tells you when you are over the instruction budget rather than over the
token budget.

---

## 10. Budgets and targets

| Artifact | Hard limit | Target | Notes |
|---|---|---|---|
| Root `AGENTS.md` / `CLAUDE.md` | 200 lines (smell threshold) | **≤ 150 lines / ~1.5K tokens** | one real refactor in the wild: 598 → 149 lines |
| Nested/scoped rule file | 100 lines | ≤ 60 lines | only what's specific to that subtree |
| `SKILL.md` body | 500 lines | ≤ 200 lines | above 500, split |
| Skill `description` | 1024 chars | 150–400 chars | what + when + trigger vocabulary, third person |
| Skill `name` | 64 chars, `[a-z0-9-]` | gerund form (`compressing-docs`) | no reserved vendor words |
| Reference file | none | add ToC above 100 lines | T2, loaded on demand |
| Reference depth | — | **1 level from entry file** | deeper = partial reads |
| Always-on imperatives | — | **≤ 40** | count them, don't estimate |
| NEVER rules | — | **≤ 7** | more than that and none of them are salient |
| Examples per concept | — | 2–3, diverse | boundary cases beat typical ones |
| Distinct sections | — | ≤ 10 | observed median in developer-written files: ~10 |

---

## 11. The pipeline

```
INPUT: target file(s), repo root, optional agent transcripts

STAGE 0  BASELINE
  count tokens, lines, imperatives, sections, links, examples
  extract INVARIANT SET (commands, paths, symbols, versions, error strings, URLs)
  snapshot original (git, or .orig copy)
  if transcripts available: mine violated rules + failure modes

STAGE 1  DETECT           # cheap, mechanical, explainable
  lint_leakage        <- cross-check rules against ruff/eslint/prettier/tsconfig/editorconfig configs
  redundancy          <- cross-check claims against README, docs/, code comments, other rule files
  skill_leakage       <- sections used by <20% of tasks / gated on a rare path
  blind_references    <- links with no "what/when" pitch
  conflicts           <- (scope, subject) triples with incompatible directives
  fossilization       <- git log: single commit, or last-modified >> code churn
  bloat               <- lines > 200, sections > 12, nesting > 2
  ambient             <- amnesia probe (§8.6) with a cheap model
  REPORT + await approval for anything CONFLICTING or ambiguous

STAGE 2  RESTRUCTURE      # move before you shrink
  demote rarely-used sections   -> reference/*.md  (+ pitched link)
  promote deterministic rules   -> scripts/*.py | CI | linter config
  replace payloads              -> pointers (path::symbol, rg pattern, command)
  scope-local rules             -> nested AGENTS.md
  order: invariants(stable) -> hot rules -> reference -> volatile

STAGE 3  REWRITE          # per-section budgets from §8.7
  apply T1..T20
  reverse chain-of-density loop, stop when a directive would be lost

STAGE 4  ENCODE           # micro-optimizations, §7
  ASCII punctuation, flatten nesting, drop decoration, table-ify repeats

STAGE 5  REORDER
  most-violated rule first; ≤5-line non-negotiables recap last

STAGE 6  VERIFY
  identifier diff -> fact recall -> rule enumeration -> idempotency
  optional: behavioural A/B on 3-5 tasks

STAGE 7  EMIT
  compressed file(s)
  AGENTS.notes.md   (removal ledger, not agent-loaded)
  report: before/after tokens, lines, imperatives; per-item rationale; open questions
```

**Hard constraints on the compressor:** it may **delete, merge, reorder, relocate and re-encode**.
It may **not add facts**. Any new claim must be flagged, not written.

---

## 12. Skill blueprint

### 12.1 Layout

```
compress-llm-documentation/
├── SKILL.md                      # navigation + the workflow; keep ≤ 200 lines
├── reference/
│   ├── smells.md                 # 6 smells + detection heuristics + fixes
│   ├── transformations.md        # T1..T20 with before/after
│   ├── budgets.md                # §10 table + per-section compression ratios
│   ├── verification.md           # the 5 tests, with prompts
│   └── evidence.md               # citations, so the skill can justify a deletion
└── scripts/
    ├── measure.py                # tokens, lines, imperatives, sections, links -> JSON
    ├── extract_identifiers.py    # invariant set -> JSON
    ├── detect_lint_leakage.py    # rules vs ruff/eslint/prettier/tsconfig/editorconfig
    ├── detect_conflicts.py       # (scope, subject, directive) triples -> conflict report
    ├── check_references.py       # paths exist, symbols resolve, links have pitches
    └── verify.py                 # identifier diff + idempotency; exit non-zero on loss
```

Scripts are T3: they cost nothing to hold and return a few lines. That is the whole point.

### 12.2 Draft `SKILL.md`

```markdown
---
name: compress-llm-documentation
description: Compresses and rewrites documentation that agents read - AGENTS.md, CLAUDE.md,
  SKILL.md, rule files, memory files - reducing tokens while preserving every command, path and
  directive. Use when a context file is bloated, slow, contradictory, stale, or exceeds its line
  budget, or when the user asks to shorten, condense, optimize or clean up agent instructions.
---

# Compressing LLM documentation

Compress by MOVING and DELETING first; rewrite sentences last. Never alter an identifier.

## Non-negotiables

- NEVER change a command, path, flag, symbol, env var, version, error string or URL.
- NEVER add a fact that is not in the source. Flag gaps instead.
- NEVER auto-resolve contradictory rules. Report them and ask.
- ALWAYS snapshot the original before editing.
- ALWAYS report before/after tokens, lines and imperative count.

## Workflow

Copy this checklist and track progress:

- [ ] 1. Baseline: run `scripts/measure.py` and `scripts/extract_identifiers.py`
- [ ] 2. Detect: run detectors; read reference/smells.md; report findings, get approval
- [ ] 3. Restructure: relocate to reference/, promote to scripts/, replace payloads with pointers
- [ ] 4. Rewrite: apply reference/transformations.md at the ratios in reference/budgets.md
- [ ] 5. Encode + reorder: micro-optimize; most-violated first, non-negotiables recap last
- [ ] 6. Verify: run `scripts/verify.py`; then the recall and rule-enumeration probes
- [ ] 7. Emit: compressed files + AGENTS.notes.md removal ledger + summary report

Stop and ask the user if: identifiers would change, rules conflict, a claim looks stale but
cannot be verified, or verification fails twice on the same check.

## Budgets

Root rule file <= 150 lines. SKILL.md <= 500 lines (target 200). Always-on imperatives <= 40.
NEVER rules <= 7. References one level deep. ToC in any reference file over 100 lines.
Full table: reference/budgets.md

## What to delete on sight

Generic advice ("follow best practices"), explanations of standard tools, directory-tree
overviews, style rules already in a linter config, restated package scripts, dated conditionals,
decorative badges and ASCII art, multi-option menus. Details: reference/smells.md

## Reference

- reference/smells.md — the 6 configuration smells, how to detect each, and the fix for each.
  Read during step 2.
- reference/transformations.md — T1..T20 rewrite patterns with before/after. Read during step 4.
- reference/budgets.md — line/token/rule budgets and per-section compression ratios. Read in step 4.
- reference/verification.md — the 5 verification tests and their prompts. Read during step 6.
- reference/evidence.md — research citations for justifying a deletion to a skeptical reviewer.
  Read only if the user challenges a removal.
```

Note how the skill obeys its own rules: pitched links, one level deep, verb-first lines, a hard
non-negotiables block, an explicit stop condition, and no explanation of what Markdown is.

### 12.3 Reusable pass prompts

**Ambient-knowledge probe (per claim, cheap model, no repo context):**
```
Repo type: {stack}. Question: {question implied by claim}.
Answer in one line. If you would need to inspect the repo, answer exactly: NEED-REPO.
```
Match against the claim: agreement → delete; contradiction → keep and promote; NEED-REPO → keep,
compress.

**Reverse chain-of-density pass:**
```
Here is a documentation section and its DIRECTIVE+IDENTIFIER set.
Rewrite it ~20% shorter. Every item in the set must appear verbatim in your output.
You may delete narration, motivation, restated general knowledge, and transitions.
You may not add information. Output only the rewritten section.
```

**Conflict extraction:**
```
Extract every rule as: scope | subject | directive | modality(MUST|DEFAULT|OPTIONAL) | line.
Output TSV, one rule per line. Do not summarize, do not merge, do not editorialize.
```

**Rule-enumeration probe (verification):**
```
Read this file. List every rule you would follow while working in this repository,
one per line, in the order you would prioritize them. Do not add rules of your own.
```

### 12.4 Scoring rubric (report this to the user)

| Metric | Green | Yellow | Red |
|---|---|---|---|
| Token reduction | ≥ 40% | 15–40% | < 15% |
| Identifier loss | 0 | — | any |
| Must-keep fact recall | 100% | — | < 100% |
| Imperatives | ≤ 40 | 40–70 | > 70 |
| Lines (root file) | ≤ 150 | 150–200 | > 200 |
| Reference depth | 1 | 2 | ≥ 3 |
| Unpitched links | 0 | 1–2 | ≥ 3 |
| Unresolved conflicts | 0 | — | any |
| Idempotent on 2nd run | yes | — | no |

---

## 13. Worked example

**Before** (~210 tokens):

```markdown
## Testing Philosophy and Approach

Testing is a critically important part of our development process here at Acme. We
believe strongly that well-tested code is maintainable code, and we ask that all
contributors take testing seriously and follow industry best practices.

Our test suite is built on pytest, which is a popular Python testing framework. You
should generally try to run the tests before you push your changes, although in some
cases for very small changes this may not be strictly necessary. The command to run
tests is `pytest`. There are also some integration tests, which live in the
`tests/integration` directory, and these require Docker to be running. Note that we
prefer descriptive test names and we use snake_case for test function names.
```

**After** (~70 tokens, −67%):

```markdown
## Tests

- Run `pytest tests/unit -q` before every push.
- Integration tests need Docker: `docker compose up -d && pytest tests/integration`.
- Integration tests are slow (~4 min). Run them only when touching `api/` or `db/`.
```

What happened, and why each move is defensible:

| Cut | Class | Reason |
|---|---|---|
| "Testing is critically important…" | DECORATIVE | zero information, induces extra exploration |
| "pytest is a popular Python framework" | AMBIENT | the model knows pytest |
| "snake_case test names" | TOOL-ENFORCED | already in `ruff.toml` |
| "you should generally… although in some cases" | hedging | promoted to a hard rule (T2) |
| — | — | **added** a stop condition + a scope gate (T11, T3), which the original lacked |

The compressed version is shorter *and* more decidable: it now answers "when do I run the slow
tests?", which the original left to the model to invent.

---

## 14. Anti-pattern quick list

- Running `/init` and committing the output unread. Measurably negative in controlled tests.
- Regenerating the file with a *stronger* model and assuming it will be better. It isn't — stronger
  models did not produce better context files in the ETH study.
- Writing an overview of the directory structure. Does not help agents locate files.
- One giant file "so everything is in one place".
- Duplicating README content "for convenience".
- Restating linter rules "so the agent knows".
- Deep reference chains (`SKILL.md → a.md → b.md → c.md`).
- Bare paths with no pitch.
- Long NEVER lists (>7) — salience collapses.
- Emoji legends and box-drawing art.
- Telegraphic English to save tokens.
- Compressing the compressed output repeatedly.
- Shipping a compression without a recall test.
- Never deleting anything, only appending. Rule files must shrink sometimes.

---

## 15. Resources

**Standards & official guidance**
- AGENTS.md standard — https://agents.md/
- Anthropic, Skill authoring best practices — https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices
- Anthropic, Agent Skills overview — https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview
- Anthropic, Effective context engineering for AI agents — https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
- Claude Code memory/CLAUDE.md docs — https://code.claude.com/docs/en/memory
- llms.txt proposal — https://llmstxt.org/

**Empirical studies (the load-bearing ones)**
- Gloaguen et al., *Evaluating AGENTS.md: Are Repository-Level Context Files Helpful for Coding Agents?* (2026) — https://arxiv.org/abs/2602.11988 · code: https://github.com/eth-sri/agentbench
- dos Santos et al., *Configuration Smells in AGENTS.md Files* (2026) — https://arxiv.org/abs/2606.15828
- Chatlatanagulchai et al., *Agent READMEs: An Empirical Study of Context Files for Agentic Coding* (2025) — https://arxiv.org/abs/2511.12884
- Mohsenimofidi et al., *Context Engineering for AI Agents in Open-Source Software* (2025) — https://arxiv.org/abs/2510.21413
- *On the Impact of AGENTS.md Files on the Efficiency of AI Coding Agents* (2026) — https://arxiv.org/abs/2601.20404
- *SkillsBench: Benchmarking How Well Agent Skills Work Across Diverse Tasks* (2026) — https://arxiv.org/abs/2602.12670

**Attention, length and instruction density**
- Hong, Troynikov, Huber, *Context Rot* (Chroma, 2025) — https://research.trychroma.com/context-rot
- Liu et al., *Lost in the Middle* (TACL 2024) — https://aclanthology.org/2024.tacl-1.9/
- Jaroslawicz et al., *How Many Instructions Can LLMs Follow at Once?* (IFScale, 2025) — https://arxiv.org/abs/2507.11538 · https://distylai.github.io/IFScale/
- Sclar et al., *Quantifying Language Models' Sensitivity to Spurious Features in Prompt Design* (ICLR 2024) — https://arxiv.org/abs/2310.11324
- He et al., *Does Prompt Formatting Have Any Impact on LLM Performance?* (2024) — https://arxiv.org/abs/2411.10541

**Compression techniques**
- Jiang et al., *LLMLingua* (EMNLP 2023) — https://arxiv.org/abs/2310.05736 · https://github.com/microsoft/LLMLingua
- Jiang et al., *LongLLMLingua* (ACL 2024) — https://arxiv.org/abs/2310.06839
- Microsoft Research, LLMLingua project page — https://www.microsoft.com/en-us/research/project/llmlingua/
- Adams et al., *From Sparse to Dense: Chain of Density Prompting* (2023) — https://arxiv.org/abs/2309.04269
- Mu et al., *Learning to Compress Prompts with Gist Tokens* (NeurIPS 2023) — https://arxiv.org/abs/2304.08467

**Self-improving / evolving context**
- Zhang et al., *Agentic Context Engineering: Evolving Contexts for Self-Improving Language Models* (2025) — https://arxiv.org/abs/2510.04618
- Suzgun et al., *Dynamic Cheatsheet: Test-Time Learning with Adaptive Memory* (2025) — https://arxiv.org/abs/2504.07952

**Formats & practitioner writing**
- TOON (Token-Oriented Object Notation) — https://github.com/toon-format/toon · benchmark critique: https://www.improvingagents.com/blog/toon-benchmarks/
- GitHub, *How to write a great agents.md: lessons from 2,500+ repositories* — https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/
- mgechev, *skills-best-practices* — https://github.com/mgechev/skills-best-practices
- obra, *superpowers* (skill framework + writing-skills) — https://github.com/obra/superpowers

---

## Appendix A — Detection heuristics

```bash
# size
wc -l FILE
python -c "import tiktoken,sys;print(len(tiktoken.get_encoding('cl100k_base').encode(open(sys.argv[1]).read())))" FILE

# instruction budget
rg -c -i '^\s*[-*0-9.]*\s*(must|never|always|do not|don.t|use |run |avoid|prefer|ensure|make sure)' FILE

# hedging (candidates for T2 normalization or deletion)
rg -n -i '\b(should|may|might|consider|recommended|typically|generally|feel free|if possible)\b' FILE

# open enumerations (T10)
rg -n -i '\b(etc\.?|and so on|among others|as needed|where appropriate)\b' FILE

# generic filler (delete on sight)
rg -n -i '(best practice|clean code|maintainable|production.ready|be thorough|as appropriate)' FILE

# lint leakage candidates
rg -n -i '(indent|spaces|tabs|camelCase|snake_case|PascalCase|line length|import order|semicolon|quotes)' FILE

# blind references (link/path with no pitch on the same line)
rg -n '\[[^]]+\]\([^)]+\)|`[a-zA-Z0-9_./-]+\.(md|py|ts|json|ya?ml)`' FILE | rg -v ' — | - .*(read|when|contains|use)'

# decoration
rg -n '[│┌┐└┘├┤─╔╗╚╝═║]|!\[.*\]\(.*shields\.io' FILE
rg -nP '[\x{1F300}-\x{1FAFF}\x{2600}-\x{27BF}]' FILE

# heading depth / structure
rg -n '^#{4,}' FILE          # too deep
rg -c '^## ' FILE            # section count (target <= 10)

# staleness
git log --oneline -- FILE | wc -l         # 1 commit => Init Fossilization
git log -1 --format=%cs -- FILE           # vs code churn since then

# reference integrity
rg -o '`[a-zA-Z0-9_./-]+\.(py|ts|tsx|go|rs|md|ya?ml|json)`' FILE | tr -d '`' | sort -u | \
  while read p; do [ -e "$p" ] || echo "MISSING: $p"; done
```

## Appendix B — Pre-ship checklist

```
STRUCTURE
[ ] Root rule file <= 150 lines; SKILL.md <= 500
[ ] References exactly one level deep from the entry file
[ ] Every link has a "what + when" pitch
[ ] ToC present in every reference file over 100 lines
[ ] Rarely-used procedures live in T2 files, not in the always-on file
[ ] Deterministic rules moved to scripts / linter config / CI
[ ] Stable content at top, volatile at bottom (cache friendliness)
[ ] Forward slashes everywhere; descriptive filenames

CONTENT
[ ] No generic advice, no ambient knowledge, no directory-tree overview
[ ] No rule duplicated from README, code, tests or a linter config
[ ] No dated conditionals (legacy folded into <details>)
[ ] No unresolved contradictions
[ ] Rationale kept only where it generalizes
[ ] 2-3 diverse examples per concept, no near-duplicates

FORM
[ ] One rule per line, verb first, <= 15 words
[ ] Modality normalized to MUST / default / OPTIONAL
[ ] <= 40 always-on imperatives; <= 7 NEVER rules
[ ] Every procedure has a stop condition and a failure branch
[ ] Enumerations closed (no "etc.")
[ ] Consistent terminology throughout
[ ] Headings are the phrases an agent would grep for
[ ] Most-violated rules first; <= 5-line non-negotiables recap last

VERIFICATION
[ ] Identifier set unchanged (0 loss, 0 invention)
[ ] Must-keep fact recall 100%
[ ] Rule-enumeration probe returns the intended rule set
[ ] Second run is byte-identical (idempotent)
[ ] Behavioural A/B: same outcome, fewer steps/tokens
[ ] Removal ledger written (AGENTS.notes.md, not agent-loaded)
```
