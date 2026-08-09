# Research: Building a `compress-llm-documentation` Skill

Grounding: Cloudflare "Markdown for Agents", Anthropic context-engineering, AGENTS.md studies (Augment), GCAT-style research (arXiv:2512.02246 token-abbreviation, arXiv:2508.13666 code-whitespace), and multiple production skill-authoring guides (Agents SDK spec, Grafana/Cloudflare/Microsoft token budgets, open-source `markdown-compressor`, `infocompressor`, `text-optimizer` skills).

---

## 1. Core Mindset (the single most important shift)

**LLM docs are a hot-loaded cache, not prose.** The measurable principle from the AGENTS.md ablation study: *duplicate/human-readable content is noise* — LLM-generated context files that paraphrased existing README docs **reduced task success in 5/8 settings**, added 2.45–3.92 steps/task, +20–23% inference cost. Architecture overviews and redundant content were pure cost.

> **Implication:** compression skill should first **delete/inferable content, not rewrite it.** "LLMs know REST, JSON, try/catch — only state what is specific to THIS system." The cheapest token is the one deleted.

**Retention hierarchy** (from the same study + Anthropic compaction guidance):
- **Keep:** non-inferable facts — commands, exact tool choices (`pixi`, not `pip`), constraints, thresholds, numbers, identifiers, paths, endpoints, `NEVER/ALWAYS` rules, decision branches, edge cases, output-format specs, YAML frontmatter.
- **Cut:** motivation/filler, restated rules, hedging ("you might consider"), transitions ("now that we've covered X"), duplicate examples, decorative markdown, HTML comments, motivational anti-laziness boosters (older-model artifacts — modern models *overtrigger* on "be thorough / don't be lazy").

---

## 2. Representation Selection (biggest single lever — pick the RIGHT form)

From `infocompressor` skill — a **decision table** for choosing the densest representation:

| Data shape | Use | Token efficiency |
|---|---|---|
| Single fact | Inline text | — |
| List (no attributes) | Bullets `-` | use `-` not `1.` (5-10% savings), or comma list `a, b, c` if 3–7 short items |
| Named properties | `key: value` | dense |
| Sequence (no branching) | Numbered list | keep order |
| Sequence with branching | Mermaid flowchart | 1 diagram ≈ many sentences |
| 2+ attributes per item | **Table** | 2–4× denser than prose |

**Non-obvious counter-rule:** Markdown table *syntax* (`| col |` + alignment row) costs **~2× tokens of the same data in minified JSON**. For dense tabular payloads embedded *in prompts/personas*, prefer minified JSON; use Markdown tables for human-facing docs. (From `text-optimizer` T.1.)

---

## 3. Token-Accounting Facts (what actually moves the needle)

Measured, not folklore:
- **HTML → Markdown: ~7–10× token reduction** (Cloudflare: 15,229 → 2,110 tokens; Fern: 1,600 vs ~16,000). This is *the* largest single win for web-to-doc conversion.
- **Unicode vs ASCII glyphs:** `→ ≠ ≥ ∵ ∴ ⊃` = 2–3 tokens each (tiktoken cl100k/o200k); ASCII `-> != >=` = **1 token**. Prefer ASCII digraphs in docs fed to models. Real win is deleting words, but glyph choice matters at scale.
- **Whitespace stripping in code-in-prompt:** Java −18.7%, C++ −13.4%, C# −11.7% tokens with <1.6% quality impact (arXiv:2508.13666). **Exception:** Python (syntax-required) and Gemini (significant degradation). Only apply to embedded code blocks, not source you want to keep readable.
- **Abbreviation is a trap:** abbreviating domain/instruction terms (`authentication` → `auth`) caused **30+ point accuracy drops** (DETAIL Matters, arXiv:2512.02246) — model uses statistically-dominant meaning. Rule: **abbreviate only universally-unambiguous tech words** (config, env, args) in tables/tech contexts; **never** abbreviate domain terms, variable names, or constraint vocabulary. The `markdown-compressor` skill ships an explicit allowlist table.
- **Budget:** 5000 tokens ≈ 3750 words (Agent Skills spec). 1 token ≈ 4 chars ≈ 0.75 words.

---

## 4. Lossless vs Lossy (two modes — encode this into the skill)

The mature skills (`markdown-compressor`) split into explicit modes:

**Lossless** (safe, zero semantic change, first pass):
- Normalize whitespace (collapse blank lines, strip trailing spaces)
- Strip HTML comments / non-instruction comments
- Merge adjacent empty headers, dedupe TOC duplicating header structure
- Normalize lists/decorative rules, remove redundant emphasis (bold+CAPS+`!`)
- Remove `---` separators, consolidate bullets → flat where possible

**Lossy** (semantic, needs review — iterate per section):
- Imperative over descriptive: "Validate input" ≠ "The system should validate input"
- One expression per concept (dedupe repeated rules, keep most complete)
- Inline over nested: "Use gzip (level 6, min 1KB)" not a nested bullet spec
- Merge sections sharing >50% content
- `Delete implied knowledge` (the highest-yield lossy move)

**Enforce a compressor–reviewer loop** as the workflow: aggressive compressor agent → reviewer agent compares source vs compressed for loss → user approves diff. Set rules on *what counts as unacceptable loss*: dropped behavioral rules, removed thresholds/values, lost edge-case logic, over-generalized instructions, broken cross-references.

---

## 5. Structural Architecture (progressive disclosure = the real "compression")

**Three-level loading model** (Agents SDK spec, Grafana anatomy) — this is *the* framework for compressing whole doc trees:

| Level | Loaded | Budget | Role |
|---|---|---|---|
| 1. frontmatter | always (startup) | ~100 tokens | `name` + rich `description` with trigger phrases |
| 2. SKILL.md body | on trigger | ≤500 (soft) / ≤5000 (hard) tokens, ≤200-500 lines | router/process, hot info |
| 3. references/ + scripts/ + assets/ | on-demand only | unlimited | details, schemas, scripts |

Rules that make this *actually* compress:
- **SKILL.md is a router, not a wiki.** One-line repo/entity descriptor + commands + a map + pointers. Anything else → `references/`.
- **One level deep only.** Never `SKILL.md → a.md → b.md` (agent stops at partial read). Link everything directly.
- **Pointer style:** plain English explicit pointer "*Writing tests: @docs/writing-tests.md*" — and *verify the agent can read it*; otherwise it guesses.
- **Name files so the agent can decide** whether to open them: `form_validation_rules.md`, not `doc2.md`.
- **`## Contents` TOC** on any reference >100 lines so agent can partial-read.
- Beyond ~250 lines, **every section is a candidate for `references/`** — cheapest way to lift conciseness.
- Frontmatter `description` carries the trigger logic (it's the only part read pre-load); don't bury triggers in the body.
- **Hot/warm/cold cache model:** root = hot (inline), module-local AGENTS.md = warm (load on proximity), wiki/README = cold (reference).

**Split-by-topic over split-by-length:** one folder per domain (`references/aws.md`, `gcp.md`, `azure.md`) so only the relevant one loads. Split-by-token (`section-1.md`…) is the fallback for oversized references.

---

## 6. Writing Style Rules (grammar of compressed docs)

Compiled rules from the best real skills:
- **Facts, not narrative.** "Auth uses JWT/RS256" ≠ "The authentication system has been designed to utilise JSON Web Tokens using the RS256 algorithm."
- **Inline definition on first use:** "The gateway (API entry point handling routing and auth) forwards…"
- **Omit** transitions, intros ("This section covers…"), and after-action summaries.
- **Rearranged word order saves tokens:** "validation must run before commit" → "validate before commit."
- **`bad → good` one-liners**: `❌ x → y` is self-documenting; keep it for rules.
- **Positive framing over negation:** "Do not use markdown" → "Write in flowing prose." Models process the stated action better (this is both a compression and a reliability win).
- **Procedures over essays:** imperative rules, decision points, tiny worked examples. A skill *changes next-turn behavior*; it doesn't explain topics.
- **Match prompt style to desired output** — less markdown in the spec → less markdown in output (relevant when the doc *is itself* generating documents).

---

## 7. Non-trivial / original tips (the "why didn't I think of that" section)

1. **Same-section independence (chunk-proofing).** LLMs/RAG retrieve *single sections*, so rewrite each heading block to be a complete thought with explicit references ("OAuth 2.0 auth", not "the auth method mentioned earlier"). Compression by isolation — each chunk must survive alone. This mirrors the `DETAIL matters` finding: explicit relationships > implied ones.

2. **Pseudocode & decision tables as compressed verbs.** Convert prose procedures to terse pseudocode or `if X → Y else Z` tables. Conditional logic is **half the token count** of the equivalent descriptive paragraph and is executed more reliably. Mermaid flowcharts collapse multi-branch sequences into one block.

3. **Formulas/notation over sentences.** A formula `p_t = softmax(QK^T/√d)` is ~6 tokens vs a 25-token sentence. Keep notation, drop the explanatory prose — the model already knows it.

4. **Spend on the *leading word* / concept anchor.** A single well-chosen term that's rich in pretraining (e.g., naming a pattern) is a compressed bundle — it pulls in the model's existing schema for free. Pick the canonical term once; don't paraphrase it 5 ways.

5. **Omit actually-inferable content, keep the *why* only when it changes behavior.** "Architecture overviews don't provide effective overviews" (removal = no behavior change, lower budget). But a rationale that disambiguates *which* choice to make ("use X *because* prod is single-tenant") earns its tokens — it's decision logic, not motivation.

6. **Stale-info tax.** Duplicated narrative (README + AGENTS.md) is not just tokens — *the divergence is active misinformation* that makes the agent pick the wrong fact. Compression = DRY for agent docs. A `references.md` index layer adds indirection to force a single source.

7. **YAML frontmatter as retrieval metadata** (`scope`, `relates-to`, `last-verified`) gives instant context without reading the body — it's free "compression via metadata."

8. **Reference commit/line against code, not prose** — the source of truth pointer is the densest possible doc ("see `src/auth/token.go:42`").

9. **Compress toward worst model, then prune.** Validate the skill on the *weakest* model that will run it; if it only works on the frontier model that wrote it, it's overfit. After editing, read it with zero conversation history and delete anything that wouldn't change action.

10. **Windows of attention.** Because LLM attention is finite, the *goal* of compression is "smallest high-signal token set that maximizes target outcome." Optimize for precision of the top ~2k tokens that stay resident — not average density of a 50k file (that's just scoring the wrong thing).

---

## 8. Concrete spec for YOUR `compress-llm-documentation` skill

Phase pipeline (mirroring proven skills):
1. **Analyze** — count tokens (cl100k/o200k, `tiktoken`), detect redundancies, flag HTML.
2. **Lossless pass** — structural cleanup, safe, no review needed.
3. **Lossy pass per-section** — apply representation-selection table + style rules; aggressive.
4. **Review** — compressor vs reviewer diff, check loss-class rules (thresholds, identifiers, negative rules, cross-refs).
5. **Restructure** — progressive-disclosure split: move detail to `references/`, one-level links, TOC on >100-line refs, router front-matter.
6. **Validate** — fresh-agent test on realistic task + weakest model; verify a reference pointer resolves.

**Measurement targets to bake in:** HTML→MD ~90% reduction; lossless pass usually 5–15%; lossy 30–40% (reported by `markdown-compressor`); whitespace-strip only on non-Python code-in-prompt.

Guardrails the skill must warn about (the "don't over-compress" list): never lose negative rules, exact numbers, tool/path identifiers, edge-case branches, or frontmatter; never abbreviate domain terms; no lossy mode without a review gate.

---

### My opinion (compressed)
The single highest-leverage insight in this whole space is **inversion**: most "compression" people attempt is *rewriting* (lossy paraphrasing), but the research consistently shows the dominant win is **deletion of inferable+redundant content, and relocation via progressive disclosure**. A compressor skill that only paraphrases is doing the bottom 20% of the work. If I were building yours, the core prompt would be three filters in order: **Delete → Relocate (to references) → Rewrite (denser form)**, with a reviewer gate. And treat token-accounting facts (unicode-vs-ASCII, whitespace-stripping applicability, abbreviation hazards) as a *lookup table* in the skill, not prose rules — because representing the rules as tables *is* the demonstration of the skill's own method.

Sources: cloudflare.com/style-guide/how-we-docs/ai-consumability, augmentcode.com/guides/how-to-build-agents-md, chase-seibert.github.io/blog/2026/02/28/coding-agent-repo-native-docs.html, anthropic.com/engineering/effective-context-engineering-for-ai-agents, github skill repos (`markdown-compressor`, `infocompressor`, `text-optimizer`), agentskills.io spec, grafana + microsoft token-budget guides, buildwithfern.com, arXiv:2512.02246, arXiv:2508.13666.