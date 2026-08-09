# llm_skills

Research toward `compress-llm-documentation` — a skill that compresses and restructures
LLM-facing Markdown (`AGENTS.md`, `CLAUDE.md`, `SKILL.md`, memory and rule files) without
losing operational meaning.

The premise, drawn from the evidence in `docs/`: a well-formed compact context file
measurably beats having none, and a bloated or auto-generated one measurably loses to
having none. Compression here is a correctness optimization, not a cost optimization.

**Status:** research phase. Nothing is implemented yet.

## Repository map

| Path | What |
|---|---|
| [docs/SYNTHESIS.md](docs/SYNTHESIS.md) | The output that matters: 15 research dossiers merged into consensus findings, six genuine disagreements with reasoned verdicts, and a canonical compression pipeline |
| [docs/INDEX.md](docs/INDEX.md) | Map of the corpus — which dossier answers which question |
| [docs/EVIDENCE.md](docs/EVIDENCE.md) | Every citation, its verification status, and three known ID conflicts |
| [docs/](docs/) | 15 raw dossiers, one per authoring model, plus a prior-art survey and a link queue |
| [CLAUDE.md](CLAUDE.md) | Working instructions for coding agents. Not written for people |

## How the corpus was built

Fifteen frontier models were given the same research brief on 2026-08-09 and answered
independently. Where they agree, the agreement is convergent rather than copied — which is
what makes the agreement counts in the synthesis worth something. Where they disagree, the
synthesis records both positions rather than picking quietly.

The dossiers are LLM-authored research notes, not peer-reviewed sources. None of the 41
arXiv citations has been verified. Treat every number as a range until it has.

## Reading it

Read `docs/SYNTHESIS.md`. It is ~5.5K tokens and supersedes roughly 96K tokens of raw
dossiers, which are about 65% mutually redundant. Reach for an individual dossier only when
the synthesis or index points you at one.

## Licence

GPL-3.0 — see [LICENSE](LICENSE).
