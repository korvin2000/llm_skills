# Compressing & Optimizing LLM Documentation — Research Basis for a `compress-llm-documentation` Skill

---

## 1. Why This Matters: Token Economics & Failure Modes

Every token carries a threefold cost: computational (more operations), financial (billed by token volume), and capacity (finite context window — every superfluous token occupies space that could carry useful information).

Critical, counterintuitive finding: **badly written docs are worse than none**. Auto-generated AGENTS.md files reduce task success rates by ~3% on average across multiple agents and models, while increasing inference cost by over 20%, and unnecessary requirements make tasks harder, increasing reasoning tokens by 14–22% (ETH Zurich, "Evaluating AGENTS.md"). Also LLM-generated AGENTS.md files reduce task success in 5 out of 8 tested settings and add 2.45–3.92 extra steps per task. Root cause: LLMs generate generic, verbose instructions stating things agents already know ("write clean code"); write by hand with only project-specific conventions the agent wouldn't know.

Second failure mode — **context rot**: large context windows dilute attention across too much material, burying important instructions; performance on tasks in the middle of very large contexts degrades significantly ("lost in the middle").

**Skill rule #1 derived**: compression = *deletion of the known* first, *rephrasing* second. The highest-ROI operation is removing content the base model already knows.

---

## 2. LLM-Optimized vs Human-Optimized Docs

| Dimension | Human docs | LLM docs |
|---|---|---|
| Audience assumption | Novice reader | Default: model is smart. If removing a sentence would not confuse a competent reader, remove it |
| Redundancy | Repetition aids learning | Redundancy = wasted tokens + attention dilution |
| Terminology | Synonyms for style | One term always ("field", never "field/box/element") to reduce cognitive load |
| Ordering | Narrative flow | Position-weighted: agent reads top-to-bottom, earlier lines carry more weight — put must-never-do rules at the very top |
| Explanations | Prose paragraphs | Executable commands, decision tables, 3–10-line code snippets |
| Motivation | Marketing intros, "why this project is great" | Zero. But keep *reasoning behind rules*: if writing ALWAYS/NEVER in all caps, reframe — explain reasoning so the model makes better judgment calls; LLMs respond better to reasoning than rote instructions |
| Format overhead | HTML/PDF/rich | Markdown: converting to Markdown reduces token count by an average of 87.5% vs HTML; Markdown for readability/token efficiency, XML when strict sectioning or deep nesting is required |
| Time references | "As of 2025..." | Avoid time-sensitive phrasing that dates the doc; put legacy info in an "old patterns" appendix |

Token cost of Markdown syntax itself is negligible: headings have minimal token cost with significant structural benefit; bullet prefixes cost 1 token; bold/italic adds 2–4 tokens per phrase; for a 1000-word doc, formatting adds ~50–150 tokens (5–15%) — small cost for structural clarity. **Opinion**: don't micro-optimize syntax characters; optimize *content selection and structure*. Syntax golf is a rounding error vs. deleting a redundant paragraph.

---

## 3. Structural Rules (What Works, Evidence-Backed)

From GitHub's analysis of 2,500+ AGENTS.md files: provide a specific job/persona, exact commands to run, well-defined boundaries, and clear examples of good output. Concretely:

1. **Commands before prose.** The most effective AGENTS.md files lead with commands rather than explanations: setup first, testing second, deployment third, debugging last.
2. **Decision tables kill ambiguity.** When a codebase has 2–3 reasonable ways to do something, decision tables force the choice up front — the pattern that most directly improved convention adherence (PRs scored 25% higher on best_practices). Tables are also token-dense: one row replaces a paragraph.
3. **Few, real examples.** Short snippets of 3–10 lines from actual production code improved reuse; keep to a few relevant, non-duplicative examples — more, and the agent pattern-matches on the wrong thing. One real code snippet is worth more than three paragraphs of description.
4. **Negative rules outperform positive ones.** Negative instructions are often more effective than positive ones — they prevent the specific mistakes the agent keeps making. A compressed doc should preserve NEVER-rules preferentially over SHOULD-rules.
5. **Content inventory**: critical rules, commands, architecture notes, coding conventions, workflow rules, maintenance habits + the WHAT (tech stack, project structure — critical for monorepos) and the WHY (purpose of key components — intent, not just structure).
6. **Hierarchical splitting**: for large repos, use nested agents.md files per directory — global rules at root, package-specific rules locally; enables independent evolution with focused guidance. Note tools favor proximity (closer/nested files) and hierarchically merge parent and child rules.
7. **Mechanical checklists** for "done" criteria: be explicit about what constitutes a "ready" change; include a mechanical checklist.

**Sober expectation**: AGENTS.md guidance alone gets ~25–40% compliance from agents; the same rules enforced as runtime hooks hit closer to 95%. Implication for your skill: when compressing, flag rules that should be *migrated to hooks/linters/CI* rather than kept as prose — that's the ultimate compression (0 context tokens, 95% compliance).

---

## 4. The Compression Ladder (Lossless → Lossy)

Order operations by information-loss risk. A skill should apply them as sequential passes:

### Pass 0 — Format normalization (lossless)
- Convert HTML/rich source → Markdown (Cloudflare blog: 16,180 tokens HTML → 3,150 Markdown, 80% savings). Tooling: [MarkItDown](https://github.com/microsoft/markitdown) — markitdown-mcp turns conversion into an MCP server so an agent converts files/URLs itself mid-conversation, returning token-efficient Markdown.
- Strip badges, HTML comments, license boilerplate, changelogs, decorative separators, duplicated ToCs.

### Pass 1 — Delete the known (highest ROI, low risk)
- Remove generic advice the model was trained on (you're burning context tokens on advice the model was trained to follow by default).
- Remove motivational/marketing prose, apologies, hedging, transitions ("as mentioned above", "it's worth noting").
- Deduplicate: same rule stated in intro + body + summary → keep one instance at highest-weight position (top).

### Pass 2 — Structural transformation (near-lossless, often *gains* clarity)
- Prose paragraph enumerating options → **decision table**.
- Step descriptions → numbered list or **pseudocode block** (see §6).
- API descriptions → signature + one-line comment instead of paragraphs.
- Enforce terminology canon (one word per concept, build a glossary once).
- Fix conflicting rules — ensure one section doesn't contradict another (contradictions cost reasoning tokens, per §1).

### Pass 3 — Progressive disclosure refactor (architectural)
Split monolith → hub + spokes. See §5.

### Pass 4 — Lossy semantic compression (apply with budget control)
Academic grounding (useful for the skill's mental model, even if you compress manually):
- Evaluate importance of each token, remove least important, produce compact representation — this is what [LLMLingua](https://arxiv.org/abs/2310.05736) automates: up to 20x compression with only a 1.5-point performance drop, and notably LLMs can effectively restore compressed prompts, and prompt compression reduces generated text length.
- Key transferable ideas: budget controller assigning varying compression ratios to different parts of the prompt (→ your skill should assign per-section budgets: NEVER-rules 1.0×, commands 0.9×, background 0.3×); dynamically assign compression budgets based on importance scores — more relevant content gets lower compression ratio.
- Selective compression (keeping subsets of original text) better preserves original content and avoids hallucinations vs generative rewriting — **prefer extraction over paraphrase** for critical rules; paraphrase only background.
- SOTA tool: LLMLingua-2 uses knowledge distillation from GPT-4-compressed prompts to train a small bidirectional encoder that classifies discardable tokens; currently SOTA for lossy compression. Repo: [github.com/microsoft/LLMLingua](https://github.com/microsoft/LLMLingua). **Opinion**: token-drop compression suits RAG payloads and transcripts, *not* durable instruction files — token-level compressed prompts may be difficult for humans to understand though highly effective for LLMs, which kills maintainability. For AGENTS.md/skills, use semantic rewriting (Passes 1–3); reserve Lingua-style compression for ephemeral context.

---

## 5. Progressive Disclosure: The Architecture of Splitting

This is the core modern paradigm, and the strongest answer to "how do you fit large context into small":

- At startup, only metadata (name and description) from all Skills is pre-loaded; SKILL.md is read only when relevant, additional files only as needed.
- Like a manual with a table of contents, then chapters, then a detailed appendix — agents load information only as needed, so the amount of context bundled into a skill is effectively unbounded.
- Hard numbers from Anthropic's authoring guide: keep SKILL.md under 500 lines, keep file references one level deep, add a table of contents to long reference files.
- Cost intuition: an 800-line SKILL.md with forms, API references, and examples costs the same whether the user asked about forms or something else entirely.
- Pointer syntax caveat (Claude-specific, non-trivial): move detailed content to references/ files with a pointer like "Read references/agent-prompt.md for full requirements" — this is an instruction to use the Read tool, NOT an @ import; @ imports only work in CLAUDE.md, not SKILL.md.
- **Scripts as ultimate compression**: utility scripts execute via bash without loading their contents into context — only the script's output consumes tokens. Deterministic logic (validation, formatting, data lookup) → move from prose to executable script. Near-zero token cost, 100% reliability.
- Filesystem as external memory: offload context to the filesystem — Manus writes old tool results to files, Cursor offloads tool results and trajectories, letting the agent read them back if needed; this addresses the concern that compaction/summarization loses useful information.
- When NOT to split: if the entire task context fits in 5–10% of the window, front-loading is simpler; progressive-disclosure overhead only pays off when the alternative degrades quality, and tasks with high interdependency across reference materials are hard to phase — compaction may work better than strict progressive disclosure.
- Discovery risk: progressive disclosure fails if the agent never discovers the skill — keep discovery names obvious. The `description` frontmatter is the load-bearing element: it should include both what the skill does and when to use it, written in third person (it's injected into the system prompt; inconsistent POV causes discovery problems).

**Hub-and-spoke pattern for your skill's output:**

```
AGENTS.md (hub, <150 lines)         # always loaded: NEVER-rules, commands, decision tables, file map
├── docs/architecture.md            # loaded on architectural tasks
├── docs/conventions/{lang}.md      # loaded per-language
├── docs/legacy-patterns.md         # appendix, rarely loaded
└── scripts/validate.py             # executed, never read
```

---

## 6. Non-Trivial / Original Techniques (Brainstorm + Opinion)

**a) Pseudocode > prose for procedures.** A 12-line pseudocode block with branch conditions replaces ~400 tokens of prose, is unambiguous, and matches the model's code-heavy training distribution. Use it for workflows with conditionals:

```
if PR touches /api: run contract tests; require schema-diff review
elif migration present: NEVER auto-apply; generate plan → human gate
else: standard flow
```

**b) Decision tables as compressed if/else trees** — evidence in §3. One table row ≈ 15 tokens vs ~60 tokens of prose per decision.

**c) Grep-ability as a design constraint.** Agents locate context via search: most tools combine search approaches; text-based search (grep, ripgrep) provides fast exact matches. So compressed docs must keep exact, stable keywords (error strings, env var names, command names) verbatim — never paraphrase identifiers. This is a *constraint on compression*: some strings are load-bearing anchors.

**d) Validator loops instead of descriptive rules.** Wire an explicit loop: produce output, run a validator, fix and revalidate; validator can be a script or a document; loop terminates only when validation passes. Converts N lines of style prose into 1 line + a script.

**e) Formulae and symbolic notation.** For quantitative rules, `retry_delay = min(2^n * 100ms, 30s)` beats a paragraph. Models parse math notation fine; it's dense and unambiguous. Similarly regex (`branch ~= ^(feat|fix)/[a-z0-9-]+$`) compresses naming-convention prose.

**f) Avoid images for critical content.** Counter to the brainstorm idea: image links are anti-compression for text agents — many harnesses can't render them, and vision tokens are expensive. Use ASCII diagrams / Mermaid (text-native, grep-able) for architecture sketches. Exception: pixel-precise UI specs.

**g) Multilingual compression — don't.** Some languages (Chinese) encode more semantics per token, but mixing languages degrades instruction-following reliability and maintainability. English is the highest-reliability instruction language for current SOTA models. Not worth ~10-15% token savings.

**h) Calibrate terseness to the weakest target model.** What reads as crisply concise to Sonnet may be too terse for Haiku; aim for the detail level that works for the weakest model you support. Your skill should take a `target_model_tier` parameter.

**i) Self-containment for subagents.** Subagents start with blank context and don't inherit the parent conversation — repeat critical rules in every agent prompt and include all necessary context inline. Compression must not create cross-file dependencies for content destined for subagent prompts; also each skill should be fully self-contained (no cross-skill dependencies).

**j) Memory files: notes as persistent state, history as ephemeral.** Structured note-taking: the agent writes notes persisted outside the context window, pulled back later — persistent memory with minimal overhead. For compressing memory files: keep recent turns verbatim (recency bias works in your favor) while compacting older context; maintain a working-memory document with key facts, treating conversation history as ephemeral and notes as persistent state. Compaction quality bar: summarize preserving architectural decisions, unresolved bugs, and key implementation details. Also auto-memory is not a substitute for checked-in team conventions; it's personal/session glue.

**k) "Unhobbling" as a compression pass.** Delete constraints that once prevented worst cases but now create conflicting instructions and wasted tokens — as models improve, old defensive rules become negative-value. Your skill should flag rules like "always confirm before X" for review against current model capability. Caveat: some domains still need hard rules (finance, medicine, regulated deploys).

**l) Token-count instrumentation.** Emit before/after token counts per section (tiktoken/anthropic tokenizer) and a compression report. Non-obvious: also measure *effective* cost — cached prefix tokens are cheap; frequently-changing sections break prompt cache. Order output docs stable-content-first for cache-friendliness (prompt caching mitigates progressive-disclosure round-trip costs).

---

## 7. Anti-Patterns Registry (What Your Skill Must Detect)

| Anti-pattern | Detection heuristic | Fix |
|---|---|---|
| Generic filler ("write clean code") | Matches training-distribution advice, no project nouns | Delete ("your agent already knows this") |
| Vagueness | "Write clean code" less helpful than "Use functional components with hooks" | Rewrite concrete or delete |
| Contradictions | Rule pairs conflict across sections | Merge; keep top-positioned version |
| Buried critical rules | NEVER-rules below fold | Hoist to top (put must-never-do things at the very top) |
| Example overload | >~5 similar snippets | Keep few, non-duplicative; more causes wrong pattern-matching |
| Time-stamped phrasing | "before Aug 2025", "currently" | Move to "old patterns" appendix |
| Synonym drift | Multiple terms per concept | Canonicalize |
| Monolith SKILL.md | >500 lines | Hub + `references/` one level deep + ToC |
| Deterministic logic as prose | Step lists a script could do | Extract to script (only output consumes tokens) |
| ALL-CAPS command barking | ALWAYS/NEVER without rationale | Reframe with reasoning for edge-case judgment |
| Stale content | References to removed code/tools | Docs must evolve with the project; regular updates keep them effective |

---

## 8. Skill Blueprint: `compress-llm-documentation` Pipeline

```
INPUT: doc(s).md, target_budget_tokens, target_model_tier, doc_type ∈ {agents_md, skill, memory, reference}

1. PARSE      → AST (headings, tables, code, lists); token-count per section
2. CLASSIFY   → each block: {never_rule, command, decision, example, arch, background, filler}
3. BUDGET     → per-class compression ratio (never_rule=1.0, command=0.95,
                decision=0.9, example=0.7, arch=0.5, background=0.2, filler=0.0)
                # LLMLingua "budget controller" idea applied semantically
4. LINT       → anti-pattern registry (§7): contradictions, vagueness, staleness,
                duplication, generic-advice detector
5. TRANSFORM  → prose→table, prose→pseudocode, steps→checklist, style-rules→validator-script,
                canonical terminology pass; PRESERVE verbatim: identifiers, commands,
                error strings, file paths (grep-anchors)
6. RESTRUCTURE→ if doc_type=skill and >500 lines: hub+references/ split with Read-tool
                pointers; if agents_md in monorepo: propose nested files
                position-sort: never_rules → commands → decisions → arch → appendix
7. VERIFY     → (a) token report before/after
                (b) LLM-as-judge: "list all constraints in original vs compressed; diff"
                (c) baseline eval: run representative tasks with old vs new doc
8. OUTPUT     → compressed docs + compression_report.md + "migrate-to-hook" suggestions
```

Verification is non-negotiable — mirror the skill-testing methodology: measure whether the agent invokes correctly and whether output matches expectations via baseline comparison — realistic prompts in fresh sessions with and without; fresh sessions matter because leftover authoring context masks gaps. And build evaluations first, establish a baseline without, write minimal instructions, then iterate. Post-deploy, watch for unexpected exploration paths, missed file references, overreliance on certain sections (move that content into the main file), and ignored content (delete or re-signal it).

---

## 9. My Opinionated Summary (Priority Order)

1. **Deletion beats rewording 10:1.** Most token waste is content the model already knows or contradicting/stale rules — not verbose phrasing.
2. **Structure is compression.** Tables, pseudocode, checklists carry more bits/token than prose and simultaneously improve compliance.
3. **Architecture beats both.** Progressive disclosure (hub + on-demand references + executable scripts) makes doc size effectively unbounded at near-constant context cost.
4. **The best compression is code**: rules → validators/hooks (25-40% → ~95% compliance), procedures → scripts (tokens → zero).
5. **Never compress**: NEVER-rules, exact commands, identifiers/grep-anchors, the single best example per pattern.
6. **Always verify** with fresh-session A/B evals; compression without evals is guessing.

## Key Resources

| Resource | Why |
|---|---|
| [Anthropic: Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) | Canonical: 500-line rule, progressive disclosure, description writing, eval loop |
| [Anthropic: Effective context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) | Compaction, note-taking, tool-result clearing, progressive disclosure theory |
| [Anthropic: Agent Skills engineering](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills) | Progressive disclosure as core design principle |
| [GitHub Blog: lessons from 2,500 agents.md repos](https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/) | Empirical patterns: persona, commands, boundaries, examples |
| [philschmid.de: Writing a Good AGENTS.md](https://www.philschmid.de/writing-good-agents) | ETH Zurich eval data; WHAT/WHY framing |
| [Augment Code: good AGENTS.md](https://www.augmentcode.com/blog/how-to-write-good-agents-dot-md-files) | Decision tables + real-snippet evidence with metrics |
| [hboon.com: AGENTS.md that works](https://hboon.com/how-to-write-an-agents-md-that-actually-works/) | Position weighting, negative instructions |
| [LLMLingua family (MSR)](https://github.com/microsoft/LLMLingua) | Budget controller, token-importance, LLMLingua-2 classifier — transferable mental models |
| [Claude Cookbook: memory, compaction, tool clearing](https://platform.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools) | Implementation of compaction primitives |
| [MarkItDown](https://github.com/microsoft/markitdown) | Any-format → token-efficient Markdown, incl. MCP server |