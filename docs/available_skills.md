Yes. I searched specifically for **Agent/Claude skills that compress Markdown, CLAUDE.md/AGENTS.md, context, or agent documentation**. There are several useful starting points, but I did **not** find one dominant, highly-starred skill that exactly does what you described earlier: *semantics-preserving optimization of technical LLM-facing Markdown documentation*. The best existing pieces cover different parts of that problem.

### Best candidates to study

| Candidate                                                                             |                    Popularity | Best for                                         | My assessment                                |
| ------------------------------------------------------------------------------------- | ----------------------------: | ------------------------------------------------ | -------------------------------------------- |
| **`caveman-compress` — JuliusBrussee/caveman**                                        |           ~65.5k GitHub stars | Direct compression of `CLAUDE.md`, memory/docs   | **Best practical starting implementation**   |
| **Context Compression / Context Optimization — Agent-Skills-for-Context-Engineering** |                   ~16k+ stars | Theory + disciplined context/token optimization  | **Best methodology source**                  |
| **SkillReducer**                                                                      | research, 55k skills analyzed | Compressing `SKILL.md` while preserving behavior | **Best scientific basis**                    |
| **`optimize-agent-docs` — petekp/agent-skills**                                       |                 only ~3 stars | Retrieval-oriented restructuring of agent docs   | Interesting architecture, not mature/popular |
| **fedir Claude compression gist**                                                     |          gist, not major repo | Extreme 80–180-token skill/agent compression     | Useful ideas, too aggressive alone           |

## 1. `caveman-compress` — most useful existing implementation

This is the strongest direct match I found. The `caveman` repository currently has roughly **65.5k GitHub stars**, and includes a dedicated `caveman-compress` skill. ([GitHub][1])

It explicitly supports:

```text
CLAUDE.md
todos
preferences
.md
.txt
.rst
.tex
...
```

Its workflow is roughly:

```text
input document
→ detect type
→ LLM compress
→ deterministic validation
→ targeted repair if validation fails
→ preserve original backup
```

The project reports around **59.6% reduction** on one documented `CLAUDE.md` example. It validates headings, code blocks, URLs, paths, and bullets rather than blindly accepting compressed output. ([GitHub][2])

Installable skill:

```bash
npx skills add juliusbrussee/caveman --skill caveman-compress --agent claude-code
```

### What I would steal from it

* original → compressed + backup architecture;
* deterministic post-compression validation;
* targeted repair instead of regenerating everything;
* file-type filtering;
* explicit path/URL/code preservation;
* measurable before/after token reduction.

### What I would **not** copy

Its main optimization is essentially **“caveman language”**. That is clever for token reduction, but too aggressive for your intended `compress-llm-documentation` skill.

For technical documentation I would preserve:

```text
normative meaning
conditions
precedence
exceptions
MUST/SHOULD semantics
identifiers
commands
examples that disambiguate behavior
```

rather than primarily converting prose into fragments.

So: **excellent implementation skeleton, insufficient semantic model.**

---

## 2. Agent Skills for Context Engineering — best conceptual foundation

`muratcankoylan/Agent-Skills-for-Context-Engineering` is currently around **16k+ GitHub stars** and contains separate:

* `context-compression`
* `context-optimization`
* `context-degradation`
* `filesystem-context`
* `memory-systems`

skills. ([GitHub][3])

For your use case I would study **both**:

```text
context-compression
context-optimization
```

The former makes an important distinction:

> optimize **tokens per completed task**, not merely tokens per request.

That matters. Excessively compressed documentation may save 30% input tokens but make an agent repeatedly reopen files or misunderstand rules, increasing total consumption. ([Agent Skills][4])

`context-optimization` recommends prioritizing:

1. stable/cache-friendly context;
2. observation masking;
3. compaction;
4. partitioning/progressive disclosure.

It explicitly treats compression as **lossy** and emphasizes measurement rather than maximizing compression ratio. ([GitHub][5])

For your planned skill, this is arguably more important than Caveman's writing style.

---

## 3. SkillReducer — probably the most relevant research

This is especially interesting for what you are building.

The 2026 **SkillReducer** paper analyzed **55,315 public agent skills** and reports:

* > 60% of skill-body content was non-actionable;
* **48% description compression**;
* **39% body compression**;
* while functional quality actually increased by **2.8%** in their evaluation. ([arXiv][6])

The architecture is almost exactly what I would recommend for your skill:

```text
Stage 1
routing/description optimization

Stage 2
body classification
→ actionable core
→ supplementary knowledge
→ progressive disclosure
→ faithfulness validation
```

The important idea is **not merely rewriting sentences shorter**.

It first asks:

```text
Does the LLM need this instruction every invocation?
```

Then:

```text
Core behavioral rule?       → keep in SKILL.md
Conditional detail?         → references/
Example/reference material? → externalize
Redundant explanation?      → remove
Base-model knowledge?       → usually remove
```

This is substantially more sophisticated than ordinary prose compression.

For your `compress-llm-documentation` skill, **SkillReducer should probably be the main conceptual foundation**.

---

## 4. `optimize-agent-docs`

This skill is almost exactly in your domain:

> retrieval-optimized knowledge layer over `.claude`, `.codex`, `.cursor`, `.aider` documentation.

It builds manifests, task-context mappings and dense compiled artifacts instead of treating every Markdown file as permanently loaded context. ([Elite AI Tools][7])

The downside: it has only around **3 stars**, so I would treat it as an interesting design prototype rather than established practice.

Its strongest idea is:

```text
don't just compress individual files
→ redesign information retrieval
```

For example:

```text
AGENTS.md                small routing/core rules
docs/
  architecture.md       loaded when architecture involved
  testing.md            loaded when testing involved
  migrations.md         loaded only for migrations
```

This can save far more tokens than shaving another 15% off sentences.

---

## 5. Extreme compression gist

There is also a recent Claude Code compression recipe targeting roughly **80–180 tokens** per agent/skill and claiming 80–95% reductions. Its rules are essentially:

* role: one sentence;
* workflow: 3–4 steps;
* retain hard constraints;
* retain high-signal heuristics;
* remove explanation/repetition/base-model knowledge. ([Gist][8])

Useful as an **aggressive compression mode**, but I would absolutely not make it the default for complex technical skills.

---

# What I would build from these

I would **not fork a single one**. I'd combine three sources:

```text
Caveman
    ↓
implementation + validation mechanics

SkillReducer
    ↓
semantic classification + progressive disclosure

Context Engineering skills
    ↓
quality/token trade-off + context architecture
```

Your resulting compressor could have approximately:

```text
INPUT
  AGENTS.md / CLAUDE.md / SKILL.md / memory.md / technical Markdown

1. ANALYZE
   classify every information unit:
   MUST / workflow / heuristic / reference / rationale /
   example / duplicate / base-knowledge / stale

2. DEDUPLICATE
   semantic rather than lexical duplication

3. RESTRUCTURE
   core always-needed rules → primary file
   conditional/deep material → references/
   reusable facts → shared docs

4. COMPRESS
   shorter syntax
   imperative forms
   remove rhetorical prose
   collapse repeated qualifiers
   compact matrices/lists where appropriate

5. VERIFY
   identifiers preserved
   commands/paths preserved
   MUST/NOT semantics preserved
   precedence preserved
   exceptions preserved
   cross-links valid

6. MEASURE
   tokens before/after
   information/rule retention
   optionally behavioral eval

OUTPUT
   compressed files
   token delta
   removed/moved-content report
   semantic-risk report
```

That is considerably stronger than just “rewrite this Markdown shorter.”

## Most important finding

The SkillReducer result supports something you've been converging on independently:

> **Compression should primarily remove non-actionable/context-inappropriate information, not merely shorten English sentences.**

A 500-token document containing only relevant rules can outperform a 300-token document that compressed away distinctions the agent needs. Their reported **39% body reduction with +2.8% functional improvement** is particularly interesting evidence for this “less but higher signal” approach. ([arXiv][6])

Also, the official Agent Skills format itself supports exactly this architecture: a small `SKILL.md` plus optional `scripts/`, `references/`, and other resources, enabling progressive disclosure rather than injecting everything at once. ([GitHub][9]) Claude Code likewise states that a skill body is loaded only when used, making supporting reference material cheap until required. ([Claude][10])

### My ranking for your purpose

**Start by studying these in this order:**

1. **SkillReducer paper** — algorithm/design.
2. **`caveman-compress`** — working implementation and validation.
3. **`context-optimization` + `context-compression`** — optimization policy.
4. **`optimize-agent-docs`** — retrieval/progressive-disclosure ideas.
5. Extreme compression gist — only for ideas about an optional `--aggressive` mode.

If your goal is specifically to create a high-quality **`compress-llm-documentation` skill**, I would not use Caveman unchanged; **SkillReducer + Caveman mechanics is the strongest starting combination I found.**

[1]: https://github.com/juliusbrussee/caveman?utm_source=chatgpt.com "GitHub - JuliusBrussee/caveman: 🪨 why use many token when few token do trick — Claude Code skill that cuts 65% of tokens by talking like caveman · GitHub"
[2]: https://github.com/JuliusBrussee/caveman/blob/main/caveman-compress/README.md?utm_source=chatgpt.com "caveman/caveman-compress/README.md at main · JuliusBrussee/caveman · GitHub"
[3]: https://github.com/muratcankoylan/agent-skills-for-context-engineering?utm_source=chatgpt.com "GitHub - muratcankoylan/Agent-Skills-for-Context-Engineering: A comprehensive collection of Agent Skills for context engineering, multi-agent architectures, and production agent systems. Use when building, optimizing, or debugging agent systems that require effective context management. · GitHub"
[4]: https://agent-skills.md/skills/muratcankoylan/Agent-Skills-for-Context-Engineering/context-compression?utm_source=chatgpt.com "Context Compression Strategies Skill | Agent Skills"
[5]: https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering/blob/main/skills/context-optimization/SKILL.md?utm_source=chatgpt.com "Agent-Skills-for-Context-Engineering/skills/context-optimization/SKILL.md at main · muratcankoylan/Agent-Skills-for-Context-Engineering · GitHub"
[6]: https://arxiv.org/abs/2603.29919?utm_source=chatgpt.com "SkillReducer: Optimizing LLM Agent Skills for Token Efficiency"
[7]: https://eliteai.tools/agent-skills/optimize-agent-docs?utm_source=chatgpt.com "optimize-agent-docs - AI Agent skill"
[8]: https://gist.github.com/fedir/56c298ffbe0f5baa6d277adbc41fa7c9?utm_source=chatgpt.com "Compresion of SKILLs and AGENTs for Claude Code · GitHub"
[9]: https://github.com/agentskills/agentskills/blob/main/docs/specification.mdx?utm_source=chatgpt.com "agentskills/docs/specification.mdx at main · agentskills/agentskills · GitHub"
[10]: https://code.claude.com/docs/en/slash-commands?utm_source=chatgpt.com "Extend Claude with skills - Claude Code Docs"
