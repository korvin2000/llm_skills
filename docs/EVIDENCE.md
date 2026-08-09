# Evidence ledger

Every citation in the corpus, with how many dossiers rely on it and whether it has been
verified. **Nothing here is verified yet.** The dossiers are LLM-authored research notes;
citation hallucination is a known failure mode and three ID conflicts are already visible
without leaving the repo.

Use this file before quoting a number outside this project, or when a claim looks too good.

## Contents

- [Verification status](#verification-status)
- [Known citation conflicts](#known-citation-conflicts)
- [Load-bearing citations](#load-bearing-citations)
- [Single-source citations](#single-source-citations)
- [Primary vendor documentation](#primary-vendor-documentation)
- [Prior art](#prior-art)
- [Verification protocol](#verification-protocol)

---

## Verification status

| Status | Count | Meaning |
|---|---:|---|
| Unverified | **41 / 41** | No arXiv ID in this corpus has been fetched and checked |
| Cited by ≥3 dossiers | 4 | Convergent, but the dossiers may share a hallucination |
| Cited by exactly 1 dossier | 26 | Highest hallucination risk |
| Known conflicts | 3 | Two dossiers give different IDs for the same paper |

Publication months in the IDs span 2023-04 → 2026-08. IDs from `2606` onward postdate the
authoring models' training data, meaning those dossiers either had live search or
fabricated them. **Determining which is the first verification task.**

---

## Known citation conflicts

Same paper, different ID. At most one of each pair is right.

| Paper | ID A | ID B | Note |
|---|---|---|---|
| LongLLMLingua | `2310.06839` (claude, codex) | `2404.11576` (hy3) | Two dossiers vs one |
| Sclar et al., prompt-format sensitivity | `2310.11324` (claude) | `2310.11333` (m3) | Adjacent IDs — likely a digit slip in one |
| Prompt Cache (m3 §10) | `2312.00059`, attributed "Shi et al. 2024" | — | Does not match the ID commonly associated with the Prompt Cache paper (Gim et al.). Verify title and authors together |

Also worth noting, not a conflict: *Lost in the Middle* appears as arXiv `2307.03172`
(5 dossiers) and as `aclanthology.org/2024.tacl-1.9` (claude). Both are legitimate —
preprint and TACL version.

---

## Load-bearing citations

Cited by 2+ dossiers, and something in [SYNTHESIS.md](SYNTHESIS.md) depends on each.

| ID | Paper as cited | Dossiers | Carries |
|---|---|---:|---|
| `2602.11988` | Gloaguen et al., *Evaluating AGENTS.md* (ETH) | **6** | The whole "context files can hurt" thesis: −0.5–2pp success, +20–23% cost, 5/8 settings worse. SYNTHESIS §2.4, §4.1 |
| `2307.03172` | Liu et al., *Lost in the Middle* | **5** | U-shaped attention → all positional layout advice. SYNTHESIS §2.6 |
| `2310.05736` | Jiang et al., *LLMLingua* | **4** | Per-section compression budgets. SYNTHESIS §6 stage 3 |
| `2605.04426` | *Telegraph English* | **3** | The ~50% / 99.1% fidelity claim — the entire pro case in the contested §3.1. **Verify first** |
| `2310.06839` | Jiang et al., *LongLLMLingua* | 2 | Query-aware compression, budget controller |
| `2403.12968` | Pan et al., *LLMLingua-2* | 2 | Distilled token classifier |
| `2601.20404` | *Impact of AGENTS.md on Agent Efficiency* | 2 | −28.6% runtime. Directly contradicts `2602.11988`; reported as −20% (kimi) and −16.6% (mimo) — **the two dossiers disagree on the number too**. SYNTHESIS §4.1 |
| `2602.05447` | *Structured context engineering at scale* (9,649 runs) | 2 | Format has no significant aggregate accuracy effect (p=0.484). Underwrites §2.11 and the rejection of exotic formats |
| `2604.17659` | *Semantic Density Effect* | 2 | The SDE formula and the +8.4pp claim |
| `2304.08467` | Mu et al., *Gist Tokens* | 2 | Conceptual only — needs model internals, not applicable to black-box docs |

---

## Single-source citations

26 IDs asserted by exactly one dossier. Highest hallucination risk; each is the sole
support for something in §5 of the synthesis.

`2306.11644` m3 · `2309.04269` claude · `2310.08560` hy3 · `2310.11324` claude ·
`2310.11333` m3 · `2312.00059` m3 · `2404.01077` m3 · `2404.11576` hy3 ·
`2411.10541` claude · `2504.07952` claude · `2505.18011` glm52 · `2507.11538` claude ·
`2508.13666` ds_flash · `2510.04618` claude · `2510.21413` claude · `2511.12884` claude ·
`2512.02246` ds_flash · `2601.07354` grok · `2602.12670` claude · `2603.29919`
available_skills · `2604.02985` codex · `2605.10870` mimo · `2605.17304` ds_pro ·
`2605.23296` codex · `2605.29676` codex · `2606.15828` claude · `2606.19857` qwen ·
`2606.23525` codex · `2607.08032` mimo · `2607.19257` qwen · `2608.01326` mimo

Three of these carry disproportionate weight and should be verified before the others:

| ID | Claimed content | Why it matters |
|---|---|---|
| `2606.15828` | *Configuration Smells in AGENTS.md Files* — 6 smells with prevalence (Lint Leakage 62%, Context Bloat 42%, Skill Leakage 35%, …), 91/100 files affected | The detector set in SYNTHESIS §6 stage 1 is built entirely on it |
| `2603.29919` | *SkillReducer* — 55,315 skills analysed; 48% description + 39% body compression with **+2.8% quality** | The only evidence that compression can *improve* quality. The project's headline claim if true |
| `2507.11538` | *IFScale* — instruction adherence vs count; ~68% at max density; primacy peak at 150–200 | Sets the instruction budget, currently unresolved by 5× (SYNTHESIS §4) |

---

## Primary vendor documentation

Not research; these are specifications and can be treated as authoritative for *behaviour*,
though they change. Verify against the live page before relying on a limit.

| Source | URL | Used for |
|---|---|---|
| Anthropic — Effective context engineering | https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents | Attention budget, just-in-time context, compaction. Cited by 10+ dossiers |
| Anthropic — Agent Skills best practices | https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices | 500-line SKILL.md, one-level-deep refs, ToC rule, description authoring |
| Anthropic — Equipping agents with Agent Skills | https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills | Progressive disclosure as a design principle |
| Claude Code — memory / CLAUDE.md | https://code.claude.com/docs/en/memory | 200-line guidance, path-scoped rules, HTML-comment stripping, MEMORY.md limits |
| Claude — context engineering for Claude 5 | https://claude.com/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models | "Spend tokens on gotchas"; rules → judgement shift |
| OpenAI Codex — AGENTS.md | https://developers.openai.com/codex/guides/agents-md | Hierarchical discovery, nested override, 32 KiB `project_doc_max_bytes` cap |
| OpenAI Codex — skills | https://developers.openai.com/codex/skills | Description truncation under catalog pressure |
| Agent Skills specification | https://agentskills.io/specification | The three-tier load model |
| AGENTS.md standard | https://agents.md/ | The format itself |
| llms.txt proposal | https://llmstxt.org/ | Site-level LLM index convention |
| GitHub — lessons from 2,500+ agents.md repos | https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/ | Six core sections, commands-early, examples-over-prose |
| Cloudflare — AI consumability style guide | https://developers.cloudflare.com/style-guide/how-we-docs/ai-consumability/ | HTML→Markdown token accounting (the source of the disputed 7–10× figure) |
| Red Hat — AGENTS.md + Agent Skills | https://developers.redhat.com/articles/2026/07/27/standardize-project-context-agentsmd-and-agent-skills | 150-line budget, orientation tables, the litmus test |
| ASDLC — AGENTS.md spec | https://asdlc.io/practices/agents-md-spec/ | Toolchain-first principle, Pink Elephant problem |

---

## Prior art

Implementations to study before writing code. Assessments from `available_skills.md`;
star counts as reported there and **not independently checked**.

| Project | Reported scale | Take |
|---|---|---|
| `caveman-compress` (JuliusBrussee/caveman) | ~65.5k stars | Best working *mechanics*: backup → compress → deterministic validation → targeted repair. Its "caveman language" output style is exactly what SYNTHESIS §3.1 rejects. Steal the harness, not the prose model |
| Agent-Skills-for-Context-Engineering (muratcankoylan) | ~16k stars | Best *policy* source. Key framing: optimize tokens **per completed task**, not per request |
| SkillReducer (paper) | 55,315 skills | Best *architecture*: stage 1 routing/description, stage 2 body classification into actionable-core / supplementary / removable, then faithfulness validation |
| `optimize-agent-docs` (petekp/agent-skills) | ~3 stars | Immature, but the right idea: don't compress files, redesign retrieval |
| markdown-compressor (oborchers/fractional-cto) | — | Source of the lossless/lossy split and the compressor–reviewer loop that 6 dossiers repeat |
| mdcompress (dhruv1794) | — | 35 deterministic rules, 3 tiers, faithfulness audit, MCP server |
| LLMLingua family (microsoft/LLMLingua) | — | Use the budget-controller *idea*; token-level pruning itself is wrong for durable instruction files |

Raw, unannotated link queue: [reasearch_links.md](reasearch_links.md).

---

## Verification protocol

Do this before the corpus is cited anywhere outside this repo.

1. Resolve every ID against `arxiv.org/abs/<id>`. Record: resolves / 404 / resolves but
   different paper.
2. For each resolving paper, check that the **specific number** the dossier attributes to
   it appears in the abstract or paper. Attribution drift is more common than fabrication.
3. Prioritise in this order: the 4 load-bearing multi-cited IDs → the 3 disproportionate
   single-source IDs → `2605.04426` (sole support for the contested §3.1) → the rest.
4. Record outcomes **in this file**, in a `Status` column. Do not create a second ledger.
5. Any ID that fails: strike the claim it supports in SYNTHESIS.md and note the dossier
   that carried it. Do not silently delete — the failure is itself a finding about
   LLM-authored research corpora.
