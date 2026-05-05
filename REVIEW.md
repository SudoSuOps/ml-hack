# Review instructions

These rules override the default review guidance. Treat them as the highest-priority
instruction block for any review of this repo. If something here contradicts a more
generic review habit, follow these.

ML-Hack inherits the upstream `huggingface/ml-intern` review doctrine, with one
addition: **ML-Hack runs production ops on real money** (cooks consume GPU hours,
Hedera anchors cost real cents, email blasts touch a real Book of counterparties).
Reviews that miss approval-gate regressions or provenance leaks are P0 by default.

## Severity levels

Every finding carries one of three priority labels:

- **P0** — blocks merge.
- **P1** — worth fixing, not blocking.
- **P2** — informational.

Write labels as plain text (`P0`, `P1`, `P2`) in finding headers. Do not use
emoji or colored markers. Use judgment on what belongs at which level.

**Always P0** — these get blocked regardless of review depth:

- Approval gate bypassed (cook dispatch · Hedera anchor · email blast)
- Honey ledger write-path skipped (model output without provenance row)
- Hedera anchor missing or misrouted (wrong HCS topic)
- System prompt loses the contamination side-gate guidance
- Recipe deviation without justification logged
- Brand-language regression that re-introduces "Atlas-as-a-Service" or treats Hacks as generic agents

## Default bias: rigor

Reviews gate merges. ML-Hack is a small-team repo where one bad cook can cost
$50-500 of GPU time and one bad blast can burn 10K counterparties. **Default
bias is rigor, not speed.** When in doubt on a P0-class concern, investigate
further before deciding.

Rigor is not nitpicking. The P1 cap, "do not report" skip list, and verification
bar all still apply. Rigor means going deep on a small number of real concerns,
not surfacing many shallow ones.

**Hold the line on P0.** If the author pushes back on a P0 finding without a
fix that addresses the root cause, re-state the concern with citations.

For P1 and P2: if the author defers or pushes back without fixing, accept it
silently — do not re-flag on subsequent commits.

## Investigate before posting

The depth of your analysis determines the strength of your finding. For any
P0-class concern, before writing it up:

- Read the relevant callers and callees, not just the diff. Use Read and Grep
  to open files the diff doesn't touch but the changed code interacts with.
- Trace the full chain end-to-end for cook dispatch, Hedera anchor, and
  approval-gate findings. Cite each hop by `file:line`.
- Check whether the codebase already has an established pattern (this repo
  inherits patterns from `huggingface/ml-intern` upstream). If the PR
  introduces a new approach where an established pattern exists, flag that.
- Confirm the specific behavior you're claiming. "This breaks X" must be
  grounded in either the code handling X or a test exercising X.

## P1 cap

Report at most **3** P1 findings per review. If you found more, say "plus N
similar items" in the summary. If everything is P1 or below, open the summary
with "No blocking issues."

## Re-review convergence

If this PR has already received a Claude review (there is a prior review
comment by the `claude` bot), suppress new P1 findings and post only P0 ones.

## Do not report

Anything in these paths — skip entirely:

- `frontend/node_modules/**`, `**/*.lock`, `uv.lock`, `package-lock.json`
- `*.egg-info/**`, `.ruff_cache/**`, `.pytest_cache/**`, `.venv/**`
- `session_logs/**`, `reports/**`, `cook_logs/**`
- Anything under a `gen/` or `generated/` path

Anything speculative — do not post:

- "This might be slow" without a concrete complexity claim tied to a specific input size
- Hypothetical race conditions without a concrete interleaving

## Dependency PRs

For PRs whose diff is only a lockfile bump, a `pyproject.toml` change, or a
new dependency, every claim in the title or body (CVE IDs, version numbers,
behavior fixes) must match what the diff actually does. A PR that lies in its
framing is P0 regardless of whether the code change is safe in isolation.

## Verification bar

Every behavior claim in a finding must cite `file:line`. "This breaks X" is
not actionable without a line reference. If you cannot cite a line, do not
post the finding.

## Summary shape

Open the review body with a single-line tally and an explicit merge verdict,
on two lines:

```
2 P0, 3 P1
Verdict: changes requested
```

Valid verdicts:

- **Verdict: ready to merge** — no P0 findings, contributor can merge as-is once any CI passes
- **Verdict: changes requested** — at least one P0 that must be addressed before merging
- **Verdict: needs discussion** — a design-level concern the maintainer should weigh in on before the contributor iterates (use sparingly)

If it's a clean review, write `LGTM` followed by `Verdict: ready to merge`.

Then a **What I checked** bullet list — one line per major area you examined,
regardless of whether you found anything. Coverage at a glance.

## Brand discipline

ML-Hack is part of the Swarm & Bee public surface. Reviewers must catch
brand-language drift:

- "Atlas-as-a-Service" → must be "Atlas-as-a-Closer" (P0 on public-facing copy)
- "Caballerz Network LLC" → must be "Swarm & Bee LLC" with D-U-N-S 138652395
- "Donovan as Broker of Record" → must be "Founder · Family Office"
- "Hack as generic agent" → must specify Hack = junior broker (vertical-locked)
- "kill the deal" → "pass the deal" (Atlas Phase-1 Brand Correction)
