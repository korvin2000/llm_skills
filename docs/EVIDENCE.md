# Evidence ledger

Every arXiv citation in the corpus, with source count, resolution status, and claim-check
boundary. The dossiers are LLM-authored research notes; a resolving identifier does not
prove that the cited paper supports the attributed claim.

Use this file before quoting a number outside this project, or when a claim looks too good.

**Verification snapshot:** 2026-08-09. All 41 identifiers were opened on arXiv after the
independent dossier analysis. Thirty-eight match the cited topic; three resolve to
unrelated papers. Detailed result checks were concentrated on evidence that changes the
compression rules.

## Contents

- [Verification status](#verification-status)
- [Resolution ledger](#resolution-ledger)
- [Confirmed citation failures](#confirmed-citation-failures)
- [Load-bearing citations](#load-bearing-citations)
- [Single-source citations](#single-source-citations)
- [Primary vendor documentation](#primary-vendor-documentation)
- [Prior art](#prior-art)
- [Verification protocol](#verification-protocol)

---

## Verification status

| Status | Count | Meaning |
|---|---:|---|
| Resolved on arXiv | **41 / 41** | Identifier, title, authors, and abstract opened |
| Topic attribution matches | **38 / 41** | Paper is the work the dossier meant to cite |
| Confirmed wrong attribution | **3 / 41** | ID resolves, but to an unrelated paper |
| Detailed claim check | selective | Methods/results checked for load-bearing or verdict-changing claims |
| Cited by at least 3 dossiers | 4 | Convergence does not replace source verification |
| Cited by exactly 1 source file | 32 | Includes one paper cited only by the prior-art survey |

Publication months span 2023-04 through 2026-08. The recent papers are real; the observed
failure mode was mostly attribution drift rather than invented identifiers.

### Superseded pre-verification snapshot

The table immediately below is retained to show the state that triggered the audit. Its
counts and “unverified” status are historical, not current.

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

## Resolution ledger

`MATCH` means the title/topic attribution is correct, not that every number quoted in a
dossier was reproduced. `DETAILED` marks sources whose consequential methods/results were
checked beyond the abstract. Source count includes `available_skills.md` where applicable.

| ID | Cited work/topic | Sources | Status |
|---|---|---:|---|
| `2304.08467` | Gist Tokens | 1 | MATCH |
| `2306.11644` | Textbooks Are All You Need | 1 | MATCH |
| `2307.03172` | Lost in the Middle | 5 | MATCH |
| `2309.04269` | Chain of Density | 1 | MATCH |
| `2310.05736` | LLMLingua | 5 | MATCH |
| `2310.06839` | LongLLMLingua | 2 | MATCH |
| `2310.08560` | MemGPT | 1 | MATCH |
| `2310.11324` | prompt-format sensitivity | 1 | MATCH |
| `2310.11333` | cited as prompt-format sensitivity | 1 | **WRONG** |
| `2312.00059` | cited as Prompt Cache | 1 | **WRONG** |
| `2403.12968` | LLMLingua-2 | 2 | MATCH |
| `2404.01077` | efficient prompting survey | 1 | MATCH |
| `2404.11576` | cited as LongLLMLingua | 1 | **WRONG** |
| `2411.10541` | prompt formatting impact | 1 | MATCH |
| `2504.07952` | Dynamic Cheatsheet memory | 1 | MATCH |
| `2505.18011` | pseudocode instruction training | 1 | MATCH |
| `2507.11538` | IFScale | 1 | MATCH |
| `2508.13666` | code-format token efficiency | 1 | MATCH |
| `2510.04618` | Agentic Context Engineering | 1 | MATCH |
| `2510.21413` | context files in open-source software | 1 | MATCH |
| `2511.12884` | Agent READMEs | 1 | MATCH |
| `2512.02246` | DETAIL Matters | 1 | MATCH |
| `2601.07354` | symbolic instruction compression | 1 | MATCH |
| `2601.20404` | AGENTS.md efficiency | 2 | DETAILED |
| `2602.05447` | Structured Context | 2 | MATCH |
| `2602.11988` | Evaluating AGENTS.md | 5 | DETAILED |
| `2602.12670` | SkillsBench | 1 | MATCH |
| `2603.29919` | SkillReducer | 1 | DETAILED |
| `2604.02985` | prompt compression in the wild | 1 | MATCH |
| `2604.17659` | Semantic Density Effect | 2 | MATCH |
| `2605.04426` | Telegraph English | 3 | MATCH |
| `2605.10870` | decision-centric agent memory | 1 | MATCH |
| `2605.17304` | Context Codec Language | 1 | MATCH |
| `2605.23296` | parallel context compaction | 1 | MATCH |
| `2605.29676` | token-optimized agent notation | 1 | MATCH |
| `2606.15828` | configuration smells | 1 | DETAILED |
| `2606.19857` | BabelTele | 1 | MATCH |
| `2606.23525` | self-compacting agents | 1 | MATCH |
| `2607.08032` | rate-distortion memory survey | 1 | MATCH |
| `2607.19257` | prompt design at scale | 1 | MATCH |
| `2608.01326` | Context Compaction Theory | 1 | MATCH |

---

## Confirmed citation failures

All three bad IDs resolve successfully, which is why existence-only checking is
insufficient.

| Bad ID | Actual paper | Intended source | Consequence |
|---|---|---|---|
| `2310.11333` | strawberry orientation for robotic picking | Sclar et al. is `2310.11324` | m3’s format-sensitivity citation is invalid |
| `2312.00059` | photo-induced charge in a semiconductor ion trap | Prompt Cache is `2311.04934` | m3’s cache citation is invalid |
| `2404.11576` | state-space decomposition for video prediction | LongLLMLingua is `2310.06839` | hy3’s query-aware compression citation is invalid |

### Original pre-verification observations

Retained below to show how the conflicts were detected before browsing. The verdicts
above supersede the hypotheses.

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
