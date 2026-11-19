# Pre-registration: idle pauses and prompt-cache re-writes in fresh Claude Code containers

Written 19 Nov 2026, committed before any paid API call. Amendments are dated and appended; none may
change a hypothesis, threshold or outcome definition after the pilot's results are seen, except the
pause length, which the pilot fixes as stated below.

**Question.** In fresh environments, do (i) cache reads dominate a coding agent's input, (ii) idle
pauses longer than the cache lifetime cause re-writes, and (iii) does the paper's cause rule
(`tools/cache_writes.py`) label re-writes after known pauses as idle, and only those?

**Design.** Paired, within task. Tasks: SWE-bench Verified instances annotated "1-4 hours" or
">4 hours" (pool 45), sorted by id, shuffled with seed 2027; the first 24 are candidates, the first
20 whose image builds are used (`experiments/pause/tasks.json`); the 25th is the pilot. Agent: Claude
Code 2.1.326, headless (`claude -p`), model `claude-sonnet-5-5`, at most 60 turns, 45 min working
time, a per-run safety cap of USD 8 (`--max-budget-usd`), API key, one fresh container per run (official SWE-bench instance image), network only to
`api.anthropic.com`. Both conditions install the same PostToolUse hook; it logs the planned points
after tool calls 5, 15 and 25. **A:** pause 0 s. **B:** pause = cache lifetime + 60 s at each point
(at most 3). A and B of a task start together.

**Pilot (excluded).** One task, both conditions. It fixes the cache lifetime from the usage fields
(`ephemeral_5m` vs `ephemeral_1h` write tokens) and hence the pause (360 s or 3660 s), gives the
per-run cost for the budget stop, and checks from transcript timestamps that each pause gap exceeds
the lifetime.

**Outcomes** (per run with the paper's code, `tools/claude_usage.py` and `cache_writes.analyse`;
analysis `tools/pause_experiment.py`). Input = fresh + cache read + cache write. Re-write = written
tokens at positions cached at the transcript's previous request. Point re-write = re-written tokens of
requests whose gap since their transcript's previous request contains a planned point. A request is
*after an injected pause* if a pause of positive length started and ended inside that gap (hook log =
ground truth). The rule labels a re-write *idle* if its share assigned to `gap_gt_60min` plus
`gap_5_to_60min` is at least 0.5.

**Hypotheses and decision rules.**
- **H1** (composition replicates): pooled cache-read share of input has a 95% bootstrap lower bound
  above 80% in both conditions. Re-write share of written tokens is reported per condition.
- **H2** (pauses cause re-writes): mean paired B minus A point re-write tokens > 0 with 95% CI lower
  bound > 0. Secondary: whole-run paired difference in re-written tokens and in re-write USD.
- **H3** (rule validity): over re-write events (requests with re-written tokens) of valid runs, truth =
  after an injected pause, prediction = idle label; precision >= 0.90 and recall >= 0.90 (point
  estimates; CIs reported). Also reported: share of condition-A re-written tokens labelled idle;
  natural gaps longer than the lifetime (no pause injected), which count as negatives in H3 and as
  positives in a secondary precision/recall; injected pauses followed by a re-write of at least half
  the cached prefix.
- **Direct expiry check** (descriptive): for each injected pause, the first request after it, its
  cache-creation and cache-read tokens against the cached prefix before the pause (read + write of
  the previous request); a full re-write (>= 95% of the prefix written, < 5% read back) is expected.
- **Billing reconciliation** (secondary): the ledger keeps each run's UTC start/end, every provider
  request id in its transcripts and list-price cost per UTC hour and day; the author compares these
  with the console's billed usage for the key's workspace, and the difference is reported.

**Analysis.** Paired over tasks with a valid run in both conditions. Percentile bootstrap, 10,000
resamples of tasks, seed 2027. Resolved rate (official SWE-bench harness) is secondary and may be
skipped for time; final diffs are kept.

**Exclusions.** Infra failures (no API request recorded, API authentication, overload or rate-limit
errors, container failure) are re-run once; a task still failing in either condition is excluded and
reported. Agent outcomes (turn limit, timeout, unresolved) are not exclusions. Spares replace
candidates only for image build failures, before dispatch.

**Budget.** USD 150 cap. Cost per run from transcript usage at `evidence/params/anthropic_list_prices.json`
(larger of that and the CLI's own figure). No run starts if spent + max per-run cost x concurrency
(8) would exceed USD 140; runs not started are reported.

**Reporting.** All results are reported whatever they show, including failed hypotheses, the pilot,
exclusions and runs stopped by the budget.

## Amendment 1 (19 Nov 2026, before any paid API call)

Made after the pre-pilot checks and before any paid request, so no result has been seen. A static
reading of Claude Code 2.1.326 and its requests to a local stub show that an API-key session asks for
five-minute cache entries by default and one-hour entries when `ENABLE_PROMPT_CACHING_1H=1`. Five-minute
pauses test only the rule's five-to-sixty-minute branch, so a third condition tests the one-hour
branch. Everything above stays in force; thresholds are unchanged.

- **Conditions.** **A**: no pause, default (five-minute) caching, 20 tasks. **B**: three pauses of
  360 s (lifetime + 60 s) after tool calls 5, 15 and 25, five-minute caching, the same 20 tasks, paired
  with A as above. **C**: one-hour caching (`ENABLE_PROMPT_CACHING_1H=1`), one pause of 3660 s after
  tool call 10 (hook timeout 3780 s, working time 45 min plus the pause), on the first 10 of the 20
  tasks. C has no paired no-pause control. A C run that ends before tool call 10 has no pause; it is
  reported and contributes no C event.
- **C outcomes.** (a) Direct expiry check for the first request after the 61-minute pause: full re-write
  (>= 95% of the cached prefix written, < 5% read back), reported as a share of C pauses. (b) **H3-C**:
  the cause rule over C re-write events (truth = after the injected pause): precision >= 0.90 and
  recall >= 0.90, where a positive label for a C event is a share of at least 0.5 assigned to
  `gap_gt_60min` (expiry after more than an hour). (c) Composition (cache-read share of input,
  re-write share of writes) for one-hour caching, descriptive.
- **Pilot (excluded).** The pilot task in A and B. It must show five-minute writes against the real API
  (else the full run stops for review), per-run cost at most USD 4, and pause gaps longer than 300 s.
  An optional C pilot on the pilot task (excluded) runs only if the A/B pilot cost at most USD 3 per
  run; it may overlap the full run.
- **Budget order.** A/B pairs have priority: a C run starts only if the budget stop would still admit
  every A/B run not yet started at the largest per-run cost seen. Runs not started are reported.
