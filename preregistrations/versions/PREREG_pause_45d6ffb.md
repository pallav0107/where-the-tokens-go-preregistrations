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
