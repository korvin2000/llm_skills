# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Research project building `compress-llm-documentation` — a skill that compresses and
restructures LLM-facing Markdown (`AGENTS.md`, `CLAUDE.md`, `SKILL.md`, memory files, rule
files) without losing operational meaning.

Thesis, from the evidence: **the failure mode of agent context files is bloat, not the
format.** A well-formed compact file measurably beats no file; a bloated or auto-generated
one measurably loses to no file. The skill's job is to move files from the second category
to the first.

**Status: research complete, nothing implemented.** No source code, no package manifest, no
tests, no CI. `.gitignore` is a stock Node template — it does not imply a Node project.
Pick and record a stack before writing code.

## Start here

`docs/` holds 15 independently authored research dossiers, ~96K tokens, roughly 65%
mutually redundant. Reading it linearly costs half a context window and teaches you the
same twelve things fifteen times.

| Read | For | Cost |
|---|---|---|
| [docs/SYNTHESIS.md](docs/SYNTHESIS.md) | Merged findings, the 6 real disagreements with adopted verdicts, contradictory numbers, 15 single-source techniques, the canonical 8-stage pipeline, open questions. **Read before any design decision.** | ~5.5K tok |
| [docs/INDEX.md](docs/INDEX.md) | Routing table — which dossier answers which question, and each dossier's distinctive contribution | ~2.3K tok |
| [docs/EVIDENCE.md](docs/EVIDENCE.md) | 41 arXiv citations, all unverified, 3 known ID conflicts. Read before quoting any number outside this repo | ~2.6K tok |

Individual dossiers are read-on-demand. Open one only when SYNTHESIS or INDEX points at it.

## Non-negotiables

This project argues that documentation quality is a correctness problem, not a style
problem. Files here are held to the standard they describe.

- NEVER add a claim to a derived document that is not in the corpus. Mark gaps
  `<!-- GAP: ... -->` and leave them.
- NEVER copy content between SYNTHESIS / INDEX / EVIDENCE. One home per fact, link to it.
- NEVER resolve a disagreement between dossiers silently. Record both positions, the
  adopted verdict, and the reason — in SYNTHESIS.md §3, not in chat.
- Cite as `dossier.md §N` so any claim is traceable without a full-text search.
- Treat every number in the corpus as unverified. Report ranges, not point estimates.
- Update [docs/INDEX.md](docs/INDEX.md) whenever a file in `docs/` is added or renamed.

## Known issue — fix before it costs more

`docs/claude.md` collides with the `CLAUDE.md` filename on Windows' case-insensitive
filesystem, so Claude Code auto-loads all ~14.6K tokens of that dossier into any session
touching `docs/`. It is the single largest avoidable context cost in the repo — in a
project about context cost.

```bash
git mv docs/claude.md docs/claude-opus.md
```

Then update the references in `docs/INDEX.md`, `docs/SYNTHESIS.md`, and this file.

## Corpus conventions

One file per authoring model. Opaque abbreviations: `ds_pro` / `ds_flash` = DeepSeek,
`glm52` = GLM-5.2, `hy3` = Hunyuan 3, `m3` = Minimax M3, `spark1` = Spark.

Two files are not dossiers: `available_skills.md` (prior-art survey) and
`reasearch_links.md` (raw link queue; the filename typo is original — leave it, external
notes may reference it).

All dossiers are dated 2026-08-09 and were produced independently from the same brief.
Their agreement is convergent evidence, not citation chains — which is what makes the
agreement counts in SYNTHESIS.md meaningful.

## Where new work goes

Nothing below exists yet. Create it here, not elsewhere.

```
skills/compress-llm-documentation/
  SKILL.md            # <=200 lines: routing + workflow only
  references/         # T2, read on demand, one level deep, ToC above 100 lines
  scripts/            # T3, executed not read — this is where token cost goes to zero
  examples/           # before/after pairs used as evals
docs/                 # research corpus + derived documents (this stays as-is)
```

The skill must obey its own rules; a bloated `compress-llm-documentation` is a failed
demo. Budgets to enforce on our own output: `SKILL.md` ≤200 lines, references one level
deep, every link carries a `what + when` pitch.

## Commands

There is no build, test, or lint setup yet. These are the measurement commands the project
actually uses.

```bash
python -c "import os,glob;[print(f'{os.path.basename(f):<22}{os.path.getsize(f)//1024:>4}KB ~{os.path.getsize(f)//39*10:>6} tok') for f in sorted(glob.glob('docs/*.md'))]"
```

```bash
rg -c -i '^\s*[-*0-9.]*\s*(must|never|always|do not|use |run |avoid|prefer|ensure)' CLAUDE.md
```

Notes: `tiktoken` is **not installed**, so all token figures in this repo are
`bytes / 3.9` estimates — fine for ratios, wrong for absolute budgets. Install it before
publishing any measurement. Available: Python 3.13, Node 24, ripgrep 15.

## Open questions blocking implementation

Full list in [docs/SYNTHESIS.md](docs/SYNTHESIS.md) §7. The two that gate everything:

1. **Target harness set** — Claude Code only, or Codex + Cursor + Copilot too? Decides
   whether the skill emits `CLAUDE.md`, `AGENTS.md`, or both, and whether nested path
   scoping is available at all.
2. **Do the 2026 citations resolve?** 26 of 41 IDs rest on a single dossier. If the
   headline studies do not exist, the project's thesis needs new support.
