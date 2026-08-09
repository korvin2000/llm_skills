# compress-llm-documentation — state after session 2

Iteration 1 is complete and measured (see below, unchanged). Session 2 implemented all three
iteration-2 improvements iteration 1 identified, made the skill auto-discoverable, and ran one
targeted regression check rather than the full 8-run matrix (see "Session 2" at the bottom —
read that first if you only have a minute).

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
python skills/compress-llm-documentation/scripts/verify.py --work WORKDIR --after DRAFT --emit-plan WORKDIR/plan.json
python skills/compress-llm-documentation-workspace/grade.py iteration-1   # or iteration-2
```

## Session 2 — all three iteration-2 improvements shipped

Read `session.md` for the session-1 narrative; this section covers session 2 only. Kept to one
targeted regression test rather than re-running the full 8-run matrix, per the token-conservation
brief — reasoning (contrastive fixture comparison, dry-run of both scripts) did the rest of the
verification work instead of spawning more agents.

**1. Cheap early exit — done.** `analyze.py` now computes `totals.fast_path` (0 H-severity
findings, 0 HIGH budget flags, 0 conflicts) and prints a `FAST PATH` line in the digest.
`SKILL.md` stage 2 tells the model to skip the full stage-3 ledger and go straight to a short
stage-9 report when it sees this. Verified by re-running eval-1 (already-good) end to end with
the updated skill: **50,307 tokens / 122.5 s**, down from iteration-1's 90,294 tokens / 395 s for
the identical task — a 44% token cut and 3.2× faster, output still byte-identical to the source,
still 11/11 assertions (`grade.py iteration-2`). Full transcript-derived report is in
`iteration-2/eval-1-already-good/with_skill/outputs/stage9-result.md`.

While building the discriminator, contrastive testing across all four fixtures caught a real gap
before it shipped: the naive fast-path rule (H-severity + budgets only) would have wrongly fired
on the `fat-skill` fixture, whose entire defect is a 2-word, non-routing `description:` — a real
problem no existing detector flagged as HIGH. Added a new budget check (`analyze.py`, `art ==
"skill"` branch): a present-but-thin description (<40 chars or <6 words) now flags HIGH, same
severity class as a missing one. Re-checked all four fixtures after the fix —
`already-good` is the only one where `fast_path: true`, as it should be.

**2. Plan scaffolder — done.** `verify.py --emit-plan OUT.json` diffs a draft `--after` against
the snapshot, writes every genuinely-lost anchor into `released_anchors` with a
`<!-- GAP: ... -->`-style placeholder reason (merging with an existing `--plan` rather than
clobbering it), and leaves the model to supply only the reason. De-emphasised anchors (survive as
plain prose) are correctly excluded, matching `check_anchors`' own soft-loss rule. Verified locally
against the bloated-monorepo fixture: a synthetic draft that dropped `` `pnpm lint:fix` `` produced
a one-entry scaffold; filling in the reason and re-running `verify.py --plan` normally passed the
real gate (`anchor-loss: 0 unapproved losses (1 approved by plan)`). Documented in `SKILL.md`
stage 4 and `references/validation.md`.

**3. Right-size the ledger — done.** `SKILL.md` stage 3 and `references/preservation.md` now say
explicitly that ledger depth should scale with what `analyze.py` found, not with the size of the
field table — a short decision list for a small/clean file, the full per-unit table only where
findings are dense or contested. This is prose guidance, not a script gate; there's no mechanical
way to verify a model "right-sized" its own ledger, so treat this one as a soft win to watch for
in the next human review pass, not a proven number like the other two.

**Discoverability — done.** Claude Code only loads skills from `.claude/skills/`; the skill lives
at `skills/compress-llm-documentation/` per this repo's `CLAUDE.md`. Created an NTFS junction
(`.claude/skills/compress-llm-documentation` → `skills/compress-llm-documentation/`) so the skill
now appears in the available-skills list without duplicating any files. **Not committed** —
junctions don't round-trip through git or other platforms, so it's gitignored with a comment
explaining how to recreate it:
```bash
cmd /c mklink /J .claude\skills\compress-llm-documentation skills\compress-llm-documentation
```
Anyone cloning this repo on Windows needs to run that once; a non-Windows/non-junction equivalent
is an open question (symlink + `core.symlinks=true`, most likely) if this ever needs to travel.

**Regression check.** Re-ran the package's own worked example
(`examples/root-before.md` → `root-after.md` against `examples/plan.json`) through the modified
`verify.py`/`analyze.py` — still `PASS with warnings`, same warnings as before the changes.
Confirms neither script's edits altered gate behaviour for existing content.

## Not done yet

- No human review pass on the outputs. `review-iteration-1.html` is generated and unopened; the
  feedback loop in the skill-creator workflow stops there. `iteration-2/` has no viewer yet either
  — only one eval ran, and a viewer felt like more ceremony than one data point warrants.
- Evals 0, 2, 3 (bloated-root, fat-skill, conflict-injection) were not re-run with the updated
  skill — they were already correctly *not* taking the fast path (see the contrastive check
  above), so the fast-path change shouldn't touch their behaviour, but that's reasoning, not a
  measurement. If tokens allow, re-running eval-0 is the next highest-value check: it's the
  eval-1 improvement's fraternal-twin risk case (largest file, most findings) where a right-sized
  ledger (improvement 3) would show up most clearly, and it's the one where a run improvised its
  own `make_plan.py` — worth confirming `--emit-plan` actually gets used instead this time.
- Description-triggering optimisation (`run_loop.py`) never run — it needs the skill to be final.
- No behavioural A/B/C. Every run reported it as NOT RUN rather than faking it.
- `CLAUDE.md` still says "Status: research complete, nothing implemented", which is now false.
  Left unedited deliberately — it is a project-instruction file and the change should be the
  user's call.
