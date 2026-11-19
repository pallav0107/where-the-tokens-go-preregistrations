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

## Amendment 2 (19 Nov 2026, after the single-task pilot, before any analysed run)

**Why.** The single-task pilot (django__django-16631, excluded) ran as designed but finished each task in
about a minute: A in 8 requests (6 tool calls), B in 4 requests (3 tool calls), both ending normally
(`success`, `end_turn`, no permission denials, no error) with a 56- and 58-line fix in the same two
files. No setup problem was found. Five-minute writes were confirmed against the real API
(ephemeral_5m only) and a run cost about USD 0.1. The planned points (tool calls 5, 15, 25; C at 10)
were mostly not reached: B never paused. The design is therefore changed from pauses inside one task
to pauses between tasks of one continuing conversation. Thresholds, outcome definitions and the
analysis code are unchanged; "point" now means the pause between two tasks.

- **Session.** One container and one continuing Claude Code conversation that works through 4 tasks of
  the same repository and version in sequence: task 1 with `claude -p`, tasks 2-4 with
  `claude -p --continue` (a stub dry run showed that 2.1.326 keeps the same session id and transcript
  and re-sends the growing history). Before each task the repository is reset to that task's base
  commit, keeping ignored files (the editable install); one instance image serves all four because
  SWE-bench builds one environment per repository and version. The image is that of the session's most
  recent task, whose history holds the other base commits (checked at build time). Task prompts are as
  before, and tasks 2-4 start with one sentence saying the repository was reset for a new issue.
- **Sessions** (`experiments/pause/select_sessions.py`, `experiments/pause/sessions.json`, seed 2027).
  Groups = (repository, version) of SWE-bench Verified, excluding repositories with compiled extensions
  (astropy, scikit-learn, matplotlib) and the pilot instance above; eligible groups have a hard task and
  at least 4 hard or "15 min - 1 hour" tasks; groups are shuffled with the seed and stably sorted by
  number of hard tasks; the first 10 are the sessions, the 11th is the chained pilot (sympy-1.6,
  excluded). Each session takes its hard tasks first (at most 4, seeded order), filled with
  "15 min - 1 hour" tasks. The 10 sessions hold 40 tasks: 26 hard and 14 from the "15 min - 1 hour"
  bucket (7 django, 2 sympy, 1 xarray sessions). Session order is a second seeded shuffle.
- **Conditions.** **A**: no pause between tasks (zero-length markers logged), five-minute caching, 10
  sessions. **B**: a 360 s pause after tasks 1, 2 and 3, five-minute caching, the same 10 sessions,
  paired with A. **C**: one-hour caching, one 3660 s pause between tasks 2 and 3, on the first 5
  sessions in session order. Pauses are a sleep in the dispatcher between CLI invocations (a developer
  stepping away between tasks); the dispatcher logs their UTC start and end as ground truth. The hook
  only logs tool calls.
- **Caps.** At most 60 turns and 45 min per task; USD 15 per session (the CLI's `--max-budget-usd`,
  given the session's remaining amount); dispatcher stop USD 140 as before. A session stopped by its cap
  is an agent outcome. A session with an infra failure in any task is re-run once from task 1.
- **Pilot.** One chained pilot session (sympy-1.6, condition A only, about USD 1, excluded) may confirm
  against the real API that the cached prefix grows across tasks. The single-task pilot and all dry
  runs are excluded.
- **Unchanged.** H1 (per condition A and B), H2 (paired over sessions; point = inter-task pause), H3
  (A and B events), H3-C, the direct expiry check, composition, the billing reconciliation, the
  bootstrap (resampling sessions), exclusions and budget order (A/B before C).

## Amendment 3 (19 Nov 2026, after the full run, before any Amendment 3 API call)

**Why.** Two weaknesses of the analysed run. (1) H3 is near-circular: in fresh containers every re-write
event followed an injected pause (A/B: tp 30, fp 3, tn 0, fn 0; C: tp 5), so a constant "idle" rule
scores the same. The run shows the rule is sensitive to idle expiry; it never tests whether the rule
tells idle expiry apart from other invalidators, or whether its marker-first order works. (2) The direct
expiry check failed (0/35 full re-writes) because a shared head (12,784 tokens in B, 12,659 in C) was
read back after every pause; the paper calls concurrent sessions sharing system prompt and tools the
likely reason but did not test it. Nothing registered above changes: H1-H3, H3-C, the direct check and
their results stand as reported. Amendment 3 adds two arms with their own decision rules.

**Existing-data check (exploratory, computed before this amendment, no API call).** From the original
runs' transcripts: for every one of the 35 injected pauses (B 30, C 5), a request of another run whose
read + write tokens cover at least the post-pause read occurred within the cache lifetime before the
post-pause request (the nearest 2 s to 247 s before it in B, 2 s to 1,491 s in C); and the first request
of 23 of the 27 run directories (the 2 single-task pilot runs included) read cached tokens written by
another container (12,784 in A/B, 12,659 in C, or the whole first prompt when A and B of a session
started together). Necessary, not sufficient, for the concurrency explanation. `tools/pause_experiment.py`
recomputes it into the evidence file under this definition, labelled exploratory; the chained pilot
(sympy-1.6, not in the runs directory) enters through its ledger first/last request times.

**Smoke tests (excluded, no API contact).** Two dry runs against the local stub (`stub_api.py`, which now
also logs a hash of each request's tools and system blocks) on the chained-pilot image, and one dry run of
each new arm through the runner, fixed the injection mechanics below. Findings: `--model` on a
`--continue` call switches the model (transcript model ids change; the client writes a `model`
attachment); `--append-system-prompt` on `--continue` leaves the system blocks unchanged (not used);
`--disallowedTools Edit` removes Edit from the request's tool list (12 to 11 tools, tools hash changes,
system unchanged) and leaves no marker the rule reads (only a new `api-request-shape` line); adding a
local MCP server with `--mcp-config` writes `deferred_tools_delta` and `mcp_instructions_delta`
attachments (the rule's list-delta marker) while the request's tool list and system blocks stay unchanged
(the new tool is deferred and announced in the messages). No paid smoke call was made.

**Specificity arm (S).** Chained sessions as in Amendment 2 (same 10 sessions, tasks, prompts, model
`claude-sonnet-5-5`, five-minute caching, per-task caps), with invalidators injected at task boundaries.
Boundary b lies between tasks b and b+1. Injection types:
- **MS** (model switch): tasks from b+1 on run with `--model claude-opus-5-5`. Opus 5.5 because it is on
  the evidence price sheet (USD 4 / 5 / 0.2 / 20 per million input / five-minute write / hit / output
  tokens, twice Sonnet 5.5); a Haiku 5.5 id is not on the sheet (its requests would be unpriced) and
  Haiku 4.5 is an older generation. One task in four on Opus adds roughly USD 0.1-0.3 per session.
- **TL** (tool-list change): tasks from b+1 on run with `--disallowedTools Edit`.
- **MCP** (MCP server added): tasks from b+1 on run with `--mcp-config` naming a local stdio server
  (`experiments/pause/image/mcp_stub.js`: one tool, server instructions, no network).
- **P** (idle pause): the dispatcher sleeps 360 s after task b (lifetime + 60 s, as condition B).

Every change stays in force for the rest of the session, so it alters the requests only at its own
boundary. Two templates; each of the 10 sessions runs once under each (20 runs, 60 boundaries, 10 of each
type): **S1** = MCP | TL+P | MS; **S2** = TL | MCP+P | MS+P. S1 and S2 of a session start together;
concurrency 4. The session driver reads per-task `model` and `extra_args` files (`session.sh`, copied into
each S container; without those files its command is that of Amendment 2). Ground truth is the
dispatcher's log `out/inject/injections.jsonl` (boundary, types, model, arguments, dispatch time) and
`out/pause/pauses.jsonl`.

*Definitions.* Boundary request = the first request of the session's main transcript at or after the
start of task b+1 (`exit.json`). C = the cached prefix before it (the previous request's read + write).
Boundary event = a boundary request with re-written tokens > 0 (`cache_writes`, unchanged). Rule label as
in H3: idle if the share assigned to `gap_gt_60min` + `gap_5_to_60min` is >= 0.5; dominant cause = the
cause with the largest share. Truth, fixed by design: MS, TL, MS+P and TL+P are **not idle** (the injected
change alone invalidates the conversation's prefix: the other model holds no cache entry of it; the tool
list precedes every cached position); true cause `model_change` (MS, MS+P) or `config_change` (TL, TL+P).
MCP+P is **idle** (the MCP change leaves the prefix unchanged, so the pause is the cause), conditional on
manipulation check M2; MCP alone is expected to produce no event (an event there is reported, truth
`config_change`).

*Manipulation checks* (reported, not exclusions). M1: share of MS, TL, MS+P and TL+P boundaries whose
request re-wrote >= 0.5 C; expected >= 0.90. M2: share of MCP boundaries whose request read back >= 0.95 C;
if below 0.90, MCP+P events are reported but left out of every confusion count (truth ambiguous).

*Registered metrics, pass marks (point estimates, as H3) and predictions.*
- **S-spec** (primary): specificity of the idle label over not-idle boundary events (MS, TL, MS+P, TL+P),
  TN / (TN + FP); pass >= 0.90. Predicted to FAIL (about 0.75): TL+P leaves no marker the rule reads, so
  the rule falls back to the 360 s gap and labels it idle.
- **S-spec-recorded**: the same over MS and MS+P events (invalidator recorded); pass >= 0.90; predicted
  pass.
- **S-order-model**: share of MS+P events whose dominant cause is `model_change` (marker-first order);
  pass >= 0.90; predicted pass.
- **S-order-mcp**: share of MCP+P events labelled idle (the true cause); pass >= 0.90; predicted to FAIL
  (about 0): the list-delta marker is ranked first although its change did not invalidate the cache.
- **S-cause**: per-cause confusion, truth type x dominant cause, all boundary events; descriptive.
  Predicted: MS and MS+P -> `model_change`; TL -> `within_lifetime` (the change is not detected as
  `config_change`); TL+P -> `gap_5_to_60min`; MCP+P -> `config_change`; MCP -> no event.

Intervals: Wilson 95% score intervals on the counts, and percentile bootstrap resampling runs (10,000,
seed 2027). Exploratory, not decisions: the same metrics with a change of the client's
`api-request-shape` hash between the previous request and the boundary request counted as a config change
(ranked with the existing config markers); and all re-write events of the arm, not only boundary events.

**Concurrency arm (K).** Condition-B sessions (360 s pauses after tasks 1, 2 and 3, five-minute caching,
Sonnet 5.5, same caps) on the first 4 sessions in session order (django-4.0, django-4.2, django-3.0,
django-4.1; 12 pauses), run strictly one at a time with at least 420 s idle before each session (and at
least 420 s after the last request of the S arm), and no other run of this experiment on the key
meanwhile. Validity: for each K post-pause request, no request of any other run (both ledgers; transcript
times, or ledger first/last request times for runs without a directory) within the 300 s before it; a
pause that fails is excluded and reported. Use of the key outside this experiment is not visible in the
transcripts; the per-hour billing reconciliation is the check.
- **K-direct** (primary): share of valid K pauses whose first request is a full re-write as registered
  (>= 95% of C written, < 5% read back). Prediction stated in advance: if the concurrency explanation is
  right, the head is not read back. Decision: >= 0.90 -> concurrency explanation **supported**; <= 0.10
  (head still read back with no concurrent session) -> **not supported**; otherwise **inconclusive**.
- Secondary (descriptive): cached tokens read by the first request of each K session (predicted 0), and
  the post-pause read values.

**Order, budget, exclusions.** S first, then K. Separate run directory and ledger (`runs-a3/ledger.jsonl`);
hard cap USD 30 of new spend (prior spend 8.59; overall cap 150 unchanged): no run starts if Amendment 3
spend + max per-run cost (prior 1.5, else the largest seen) x concurrency > USD 30; session cap
(`--max-budget-usd`) USD 3. Infra failures re-run once from task 1 (as Amendment 2); a session stopped
early contributes the boundaries it reached; runs not started are reported. Every pass mark missed is
reported as a failure, including the predicted ones; no threshold, truth assignment or definition changes
after data. Optional, time permitting: the registered secondary outcome (resolved rate of the kept final
diffs, official SWE-bench harness), reported per condition with no hypothesis.
