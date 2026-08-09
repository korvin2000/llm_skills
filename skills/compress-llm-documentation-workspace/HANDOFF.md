# compress-llm-documentation — state at end of session 1

Iteration 1 is complete and measured. The skill is usable now. This file is what a fresh session
needs to pick it up.

## What exists

```
skills/compress-llm-documentation/
  SKILL.md                     164 lines, description 935/1024 chars
  references/                  7 files, one level deep, each with a ToC and a read-when pitch
    preservation.md            ledger, protected atoms, gaps, conflicts, injection policy
    rulebook.md                ranked catalog, the 4 gates, decision procedure, rejected list
    transformations.md         before/after patterns + per-class budgets
    harnesses.md               verified Claude Code load semantics, relocation targets
    validation.md              check ladder, probes, A/B/C, scoring rubric
    artifact-playbooks.md      per-file-type policy and target shapes
    detectors.md               finding-by-finding guidance + ripgrep fallbacks
  scripts/                     Python 3.9+, stdlib only, tiktoken used if importable
    mdlib.py                   parsing, anchors, directives, matching
    analyze.py                 baseline + all detectors + anchor set + original/ snapshot
    verify.py                  fidelity gate; exit 1 on failure
  examples/                    worked before/after pair + plan.json, verifies PASS
  evals/evals.json             4 cases, 57 assertions

skills/compress-llm-documentation-workspace/
  fixtures/                    4 eval fixtures
  grade.py                     mechanical grader for both arms
  iteration-1/                 8 runs, grading.json + timing.json each, benchmark.json
  review-iteration-1.html      static review page — open this first
```

## Decisions locked in session 1

| Decision | Choice |
|---|---|
| Levels | `safe` = Tier 1 · `medium` = + verified relocation/representation · `max` = + scripts, path-scoping, cross-file. Tier-3 experiments (telegraphic, symbolic notation, ablation) excluded at every level |
| Harnesses | Claude Code verified profile + portable fallback. Codex left as an explicit GAP, not guessed |
| Write policy | Audit → plan → apply on approval; immediate apply when the user already said "compress it" |
| Stack | Python 3 stdlib only. No tiktoken present, so all token figures are labelled ESTIMATE |

## Iteration-1 results

Assertions passed: **with_skill 57/57, baseline 54/57.** Both arms are strong — Opus handles this
task well unaided, so the skill's margin is in verified preservation, not raw capability.

Where the skill actually won:
- Removed the derivable directory tree and prose-pinned dependency list; the baseline kept both.
- Explicitly asked for a human decision on the retention conflict; the baseline surfaced both sides
  but never asked.
- Produced an audit trail every run: `plan.json`, `analysis.json`, `verify.json`, `original/`.

Cost of that rigour, tokens (with_skill vs baseline):

| Eval | with_skill | baseline | ratio |
|---|---:|---:|---:|
| 0 bloated-root | 154,288 | 57,378 | 2.7× |
| 1 already-good | 90,294 | 41,739 | 2.2× |
| 2 fat-skill | 118,997 | 49,987 | 2.4× |
| 3 conflict-injection | 130,547 | 43,394 | 3.0× |

Behavioural highlights worth keeping:
- eval-1 output is **byte-identical** to the input (md5 match) — the "already good" case works.
- eval-3 output **grew**, and the report said so plainly: waste came out, the conflict block went in.
- eval-2 reported that always-loaded context went **up** (~4 → ~131 tok) because the description had
  to become findable — the right trade, honestly disclosed.
- The eval-0 baseline hand-built an anchor verifier and it caught a regression it had just made.
  Independent confirmation that `verify.py` is the highest-value thing in the package.

## The three improvements iteration 2 should make

1. **Cheap early exit.** eval-1 burned 90K tokens to conclude "change nothing". After stage 2, if
   `analyze.py` reports no H-severity findings and budgets are clean, the skill should be allowed
   to jump straight to the report instead of building a full ledger and walking every gate.
   Biggest single win available.
2. **Bundle a plan scaffolder.** The eval-0 run wrote its own `make_plan.py` because hand-copying
   dozens of exact anchor strings into `plan.json` is error-prone busywork. Add
   `verify.py --emit-plan` that pre-fills `released_anchors` from the current losses, leaving the
   model to supply only reasons.
3. **Right-size the pipeline to the file.** A 44-line file does not need a 19-row ledger. Make the
   ledger's depth proportional to size and finding count, and say so in SKILL.md.

Smaller: several runs reported the `Write` tool blocks subagents from creating `report.md`; that is
a harness quirk of the eval setup, not a skill problem, but it cost each run a retry.

## Known-good commands

```bash
python skills/compress-llm-documentation/scripts/analyze.py FILE --out WORKDIR --repo-root .
python skills/compress-llm-documentation/scripts/verify.py --work WORKDIR --after FILE --plan WORKDIR/plan.json
python skills/compress-llm-documentation-workspace/grade.py iteration-1
```

## Not done yet

- No human review pass on the outputs. `review-iteration-1.html` is generated and unopened; the
  feedback loop in the skill-creator workflow stops there.
- No iteration 2, so no `--previous-workspace` comparison exists.
- Description-triggering optimisation (`run_loop.py`) never run — it needs the skill to be final.
- No behavioural A/B/C. Every run reported it as NOT RUN rather than faking it.
- `CLAUDE.md` still says "Status: research complete, nothing implemented", which is now false.
  Left unedited deliberately — it is a project-instruction file and the change should be the
  user's call.
- The skill lives at `skills/compress-llm-documentation/` per CLAUDE.md. Claude Code discovers
  skills under `.claude/skills/`, so it must be copied or linked there to load automatically.
