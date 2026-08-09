Compressing LLM docs without losing signal — researching token-optimal structures, AGENTS.md/memory patterns, and modern prompt compression.

# compress-llm-documentation: Research Basis for Skill

> **Core thesis:** `Markdown is not a compression format. It is a structure-preservation format`. Optimization = maximize **semantic density** (signal per token), not just minimize tokens. Context window = working memory, not hard drive.

## 1. Ground Rules (from Empirical Studies + Anthropic 2025-26)

| Rule | Evidence |
|---|---|
| **Only non-inferable content** | LLM-generated `AGENTS.md` hurts success `-0.5% to -2%` and costs `+20-23%`. Human-curated `+~4pp` only when minimal. Exclude architecture overviews agents can infer |
| **Token budget is hard** | Codex: `project_doc_max_bytes=32 KiB` default, stops loading. SKILL.md `<500 lines / <5000 tokens`. AGENTS.md split at `150-200 lines` (max observed 371) |
| **Keep root lightweight** | `Keep CLAUDE.md lightweight...spend most tokens on gotchas` |
| **Don't state obvious** | `A skill that restates what Claude would do by default adds context without value` |
| **Progressive disclosure > upfront** | `We moved verification and code review into their own skills...` + `Think of entire file system as context engineering and progressive disclosure` |

## 2. How to Write Text *for* LLM (Not Human)

**Modern shift (Claude 5 gen):** `Then: Give rules -> Now: Let use judgement`, `Then: Give examples -> Now: Design interfaces`, `Then: Put it all upfront -> Now: Use progressive disclosure`.

**Lexical / Syntactic compression pipeline:**
*   **Imperative, no hedging:** `Run`, `Use`, `Never` > `You should consider...` / `Please try to...`. Declarative invariants beat requests.
*   **Telegraph English:** Subject-Predicate-Object entity-relation statements outperform coherent prose summaries at same token budget (+13-20 F1 on QA) - use `Component --INVOKES--> Target (condition)` style.
*   **Remove glue:** Articles (`a/the`), linking verbs (`is/are`), filler (`very`, `simply`, `in order to`), politeness, apologies. See `Semantic Dense Block (SDB)` pattern: single-paragraph, no articles.
*   **Abbreviate aggressively + legend:** Define `Glossary: cfg=config, auth=authentication, req=request` once at top. Saves 15-25% on repetitive domains. LLM handles abbreviation if defined.
*   **Pseudocode > prose for procedures:** `CodeAgents` pseudocode yields concise readable plans. LLM understands control flow natively. Replace 5-sentence workflow with 3-line pseudocode.
*   **Tables > lists > prose:** For comparisons, matrices, feature flags, always tabular. `LLMs extract structured data far more reliably than prose because structured data has clear boundaries` and `Markdown table preserves row-column relationships` - critical for chunking/RAG. Prose forces parsing.
*   **One idea per bullet, no nesting >2:** Linearize. LLM attention degrades in middle of long paragraphs.

## 3. Structure Optimized for LLM Parsing

**Why markdown works:** `Headings become headings. Tables remain tables. Code blocks separated from prose. Lists are lists` + `structural cues are familiar... ## heading signals boundary as clearly as system instruction, costs 2 characters` + `hierarchical nature allows LLM to discern logical flow`.

**Ideal compressed spec layout:**
```md
# <Project/Tool>: One-line invariant
> stack: python3.12, FastAPI, pixi (not pip) // exact versions = non-inferable

## Commands [copy-paste runnable]
- test: `pixi run test` # not npm test
- lint: `pixi run lint`

## Gotchas [HIGHEST SIGNAL]
- subscriptions append-only → use max(version) not created_at
- @request_id (gateway) == trace_id (billing)

## Constraints [NEVER/ALWAYS]
- NEVER edit vendor/; NEVER rotate keys w/o #security
- ALWAYS run lint before PR

## Refs → progressive disclosure
- api.md: signatures + examples
- gotchas/auth.md: detailed flows
```

**Rules:**
*   Hierarchy: `H1`=purpose, `H2`=category, `H3`=specific. No `H4+` (flatten).
*   `Gotchas section is highest-signal content` - built from observed failures. Prioritize over tutorials.
*   `Description field is not summary, it's trigger - when to use skill`. Apply same to AGENTS.md top line.
*   `Avoid railroading Claude - give info + flexibility, not overconstrained steps`. Provide invariants, let plan.
*   Use `file:line` references to source, not duplicated code.

## 4. Progressive Disclosure Architecture (Core Compression)

**Principle:** `Progressive disclosure selectively reveals only info needed for current request` and `limiting what is added to context to minimum necessary, adding more over time`.

**Implementation for docs/skill:**
1.  **L0 - Always loaded (tiny):** Root `AGENTS.md` / `SKILL.md` frontmatter + 1-paragraph summary + index of refs. Target `<2k tokens`.
2.  **L1 - On-demand (SKILL.md body):** Loads only when skill triggered. Point to files: `references/api.md`, `assets/template.md`.
3.  **L2 - Deep refs:** Scripts, examples, full API specs. Agent `read` only when needed.

**Discovery:** Codex concatenates from root down, deeper files override, precedence = deeper nested. For Claude: `CLAUDE.md always, Skill when needed, Prompt now`. Check `~5000 tokens` budget via `wc -w *0.75`.

**Anti-monolith:** `Split project knowledge into 8KB chapters under memory/`, per-module `agent_docs/token_lifecycle.md` pattern.

## 5. Condensing Large Context → Small Without Loss

**6-stage pipeline (distilled from `llm-min.txt` SKF):**
`llm-min.txt` is `min.js` for docs: `super-condensed structured summary ... 90-95% token reduction, some >97%` + `typically 10k tokens vs 800k` + `preserves classes/methods/params/usage patterns, drops prose`.

Adapt for `compress-llm-documentation` skill:

| Stage | Action | Technique |
|---|---|---|
| **1. Gather** | Recursively scan `.md/.txt/.rst`, strip HTML/nav/footer, dedupe via hashing | Input cleaning = biggest win before compression |
| **2. Classify** | Tag each chunk: `Commands`, `Gotchas/Invariants`, `Patterns`, `Redundant` (inferable/readme-duplicate), `Narrative` | ETH finding: `removing Architecture keeps behavior same at lower cost` |
| **3. Extract entities** | LLM (Gemini 1M ctx recommended) → glossary `Gxxx` IDs, then `DEFINITIONS Dxxx`, `INTERACTIONS Ixxx` | Use small LM perplexity to score token importance if doing extractive pruning (LLMLingua pattern: `budget controller + iterative token pruning + alignment`) |
| **4. Re-express** | Rewrite each section: imperative bullets, table, pseudocode, Telegraph English `I001:G001.greet INVOKES G003.log`. Define abbrev legend. | Merge freq pairs into tokens (MedTPE idea) if extreme |
| **5. Assemble progressive** | Root = header metadata + definitions summary + `USAGE_PATTERNS U_Name.N: [Actor] ACTION (Target) -> [Result]`. Details → `references/` | Keep guideline decoder: `llm-min-guideline.md essential companion` analogue: compressed file needs legend file |
| **6. Validate** | Token count, recall check (prompt LLM to reconstruct answers from compressed vs original), `lost-in-middle` test: place critical rule early+late | If `+20% cost no gain` → over-compressed |

**Lossy vs lossless decision:**
*   **Lossless:** For API specs, commands, constraints → exact flags, no paraphrase. Use dictionary compression.
*   **Lossy:** For tutorials, explanations, history → abstractive summary. Accept 13-20 F1 tradeoff vs truncation.

## 6. LLM-Optimized vs Human-Optimized: Key Deltas

| Dimension | Human Docs | LLM Docs |
|---|---|---|
| **Goal** | Persuasion, learnability, delight | Determinism, recall, tool-use accuracy |
| **Prose** | Narrative, analogies, repetition for memory | No repetition (cost), no analogies, invariants only |
| **Examples** | 2-3 curated, pretty | 1 minimal canonical snippet >3 paragraphs+ `scripts/ libraries let Claude spend turns on composition not boilerplate` |
| **Visuals** | Screenshots, styled HTML | `Mermaid diagrams ideal: render for humans, remain text for agents`. Never embed image-only info; link to image + alt text: `![diagram](url)<!-- LLM: ... -->` |
| **Navigation** | TOC, search | `llms.txt = curated index of .md links` + `llms-full.txt = clean Markdown every code block ... regenerated automatically`. Provide both. |
| **Formatting** | HTML for rendering | Markdown: `70-88% signal vs <30% for raw PDF/PPTX`. Strip classes/JS. |
| **Error handling** | FAQ | Gotchas with `condition -> safe path` table |

## 7. Non-Trivial / Opinionated Tricks (Brainstorm)

1.  **Symbolic Metalanguage:** Use `combinatory logic / functional paradigm` symbolic compression for reasoning-heavy skills. Define operators: `=>` = implies, `!` = never, `?` = ask. `68-81% token reduction` reported for symbolic in Gemini. Example: `deploy !prod w/o smoke;ask(channel)` vs sentence.

2.  **Pseudocode as lingua franca:** Replace workflow description with Python-like pseudocode. LLMs trained on code parse it with lower ambiguity. Example:
    ```
    def review(pr):
      if pr.touches("payments/*"): run("make test-payments") # not npm test
      assert no_secrets(pr.diff)
    ```

3.  **Abbreviation + Formula:** For constraints, use formula: `cost = toks_in*3 + toks_out*15 ; budget<50k` . More precise than prose, token-cheap. LLMs evaluate correctly.

4.  **Image link deferral:** Never describe UI verbally. `See spec: ./assets/mock.html` + `Verify: run scripts/visual-check.py` . `Give Claude references as code/HTML artifact, not description or screenshot`.

5.  **Multilingual token hack:** Keep docs in English (lowest tokens/char for BPE). If team non-EN, keep legend in EN, comments in native lang optional - but core instructions EN.

6.  **Reference by code, not doc:** `Rubrics allow Claude to verify taste... via verifier agents`. Use test suite as spec instead of describing spec.

7.  **Layered markdown middle ground:** Use GFM tables, admonitions `> [!NOTE]`, collapsible `<details>` for human hide but LLM still reads - satisfies both audiences without duplication.

8.  **Store scripts and generate code** - provide `scripts/fetch_data.py` with helpers, Claude composes rather than re-describing API. `SKF` usage patterns `U_BasicGreeting.1:[User] CREATE (G001) -> [instance]` do same for docs.

9.  **Hard caps to avoid silent dropout:** `Keep files short, place critical rules early, start new sessions` due to `lost in middle`. Front-load `NEVER` rules.

10. **Output contract example:** Define JSON schema at bottom for LLM to emit structured result, not prose. Saves downstream parsing tokens: `Our preprocessed version costs less and produces weaker output before you write single prompt` tradeoff reversed.

## 8. Anti-Patterns to Enforce in Skill

*   **No duplication across tool files:** Use symlink `CLAUDE.md -> AGENTS.md`.
*   **No generic guidance** like `write clean code` → delete. `Avoid stating obvious`.
*   **No empty files** (ignored by Codex).
*   **No long sessions** accumulating context → `Use /clear between tasks` + `Use subagents for distinct phases`.
*   **No purely LLM-generated commit** → must human-review; `LLM-generated files redundant with docs agents already access`.

## 9. `compress-llm-documentation` Skill Scaffold

```
compress-llm-documentation/
├── SKILL.md  # <500 lines, <5k tokens, progressive index
├── scripts/
│   ├── token_count.py      # wc -w*0.75 + tiktoken
│   ├── dedupe.py           # hash + semantic near-dup
│   ├── compress.py         # calls Gemini/Claude to SKF/Telegraph
│   └── validate.py         # recall QA: compressed vs original
├── references/
│   ├── compression_patterns.md  # detailed D/I/U templates
│   ├── gotchas_catalog.md
│   └── llms_txt_spec.md
└── assets/
    └── template_AGENTS_min.md
```

**SKILL.md skeleton (apply rules above):**
```md
---
name: compress-llm-documentation
description: Compress/optimize AGENTS.md, CLAUDE.md, memory, Markdown skills. Use when docs exceed 150 lines, token budget bloated, or before committing agent docs.
---

# Compress LLM Documentation

## Trigger
- files >150 lines or >32KiB or user says /compress

## Pipeline
1. Audit: `scripts/token_count.py $TARGET` → report density
2. Classify: cmds/gotchas/patterns/redundant
3. Rewrite: imperative, tables, pseudocode, abbrev legend
4. Split: root <100 lines → refs/ per concern (auth, test, deploy)
5. Validate: QA recall + `file:line` check

## Rules
- Keep only non-inferable. Delete arch if exists in README/code.
- Gotchas first. NEVER rules top.
- Provide legend for abbrev. Use Mermaid not screenshots.
- Progressive disclosure: root index → refs loaded on demand.

## Refs
- Details: references/compression_patterns.md
- SKF spec: references/skf.md
```

**Compression prompt internally uses `LLMLingua coarse-to-fine`: `budget controller maintains semantic integrity under high compression, token-level iterative compression, distribution alignment` + `calculate perplexity for all tokens... verify coherence` iterative pruning.

## 10. Validation Checklist

*   Token reduction 60-95% without recall loss on 5 QA probes derived from original
*   No content that `agent can discover via codebase exploration` remains
*   Root file `wc -l <150`, total with refs accessible but not all loaded
*   All commands runnable copy-paste
*   Gotchas cover last 3 real agent failures

> **Opinion:** Most teams over-write `AGENTS.md` as human `CONTRIBUTING.md` clone. Treat it as **symbolic OS config**: minimal invariants + pointers. Let file system be your RAG. The best compressed doc is one that makes agent *fetch* less, not *read* more.

