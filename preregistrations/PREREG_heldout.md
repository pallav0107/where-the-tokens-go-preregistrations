# Pre-registration: held-out test of context x turns versus prompt size

Paper: `papers/agent2027-cost/` (AGENT 2027 short paper), Section "Does context times
turns predict cost?". Written and committed **before** any per-unit cost, feature or fit
was computed. The commit that adds this file is the registration; its hash and time are
recorded in `evidence/heldout.json` (`provenance.prereg`). Any departure from this plan is
reported in the paper and in the evidence notes as a deviation.

## Question

Does a context-size x turns descriptor predict a conversation's input cost on held-out
conversations better than a prompt-size-only descriptor?

## What was known at registration

* The aggregate snapshot already in `evidence/harness_costs.json` (no per-unit values).
* Format facts from a probe that counted fields and records only: an assistant response is
  written as several transcript lines that repeat the same message id and the same input
  usage (209,811 usage lines, 94,081 distinct message ids; `output_tokens` differs between
  repeats for 12,116 ids); every usage record carries the 5-minute / 1-hour cache-write
  split (14 records where it does not sum to the total); speed `standard`, service tier
  `standard`, inference geography `global` or not reported.
* Unit counts: 36 main-loop transcripts and 1,341 subagent transcripts; start dates by
  month. No cost, context or prompt-size value of any unit had been computed.

## Data and instrument

* Source: every Claude Code transcript (`*.jsonl`) under the operator's Claude Code projects
  directory on a local workstation, main-loop and subagent. One operator, one harness.
* Request extraction (`tools/claude_usage.py`): a request is one assistant message id with
  a `message.usage` block. Model `<synthetic>` is dropped. Repeated lines of one id become
  one request: input fields from its first line, `output_tokens` the maximum over its lines.
  Transcripts are processed in order of their first request time (ties: path order); a
  message id already attributed to an earlier transcript is skipped.
* Freeze: only requests with timestamp before **2026-11-15T00:00:00Z** are used.
* Instrument: `tokenledger-llm` **0.2.0** (PyPI), `--format anthropic`, one tokenledger
  "turn" per transcript and one "step" per request in time order. Per transcript it gives
  T = number of requests, C = peak context (`max_prefix`, the largest per-request total
  input), I = total input tokens, and P = total input of the first request.
* Prices: Anthropic list prices from the official pricing page
  (https://platform.claude.com/docs/en/about-claude/pricing), accessed 2026-11-15: per
  model, base input, 5-minute cache write, 1-hour cache write, cache hit. A transcript
  containing a model without a listed price is excluded (rule E3).

## Unit, target and exclusions

* Unit: one transcript, i.e. one conversation with its own context window (a main-loop
  session or one subagent run). Main-loop sessions alone (n about 35) are too few for a
  held-out split, so they are a reported subset, not the primary population.
* Primary target y = list-price input cost of the transcript in USD:
  sum over requests of fresh x p_in + write_5m x p_5m + write_1h x p_1h + read x p_hit. If
  a record's 5m + 1h split does not sum to its cache-write total, the remainder is priced
  as 5-minute writes.
* Exclusions, applied in this order and each counted: E1 no request before the freeze;
  E2 fewer than 3 requests; E3 a model without a listed price; E4 y = 0 or P = 0 (log
  undefined).

## Split (fixed now)

Sort eligible units by first-request time (ties: path order). Training set = the earliest
floor(0.7 n) units; test set = the rest. A temporal split: the models are fitted on the
past and scored on later conversations, including newer model versions.

## Models

Ordinary least squares on natural logs, intercept and slope, fitted on the training set
only. Both primary models have two parameters.

* **A (primary): ln y = a + b ln(C x T)**, peak context times turns.
* **B (primary): ln y = a + b ln(P)**, prompt size (first-request input).
* S1 (secondary): ln y = a + b ln(T), turns only.
* S2 (secondary): ln y = a + b ln(P x T), prompt size times turns (context that never grows).

Mean context x turns is not used: over a transcript's own requests it equals total input
tokens by construction (Eq. 1 of the paper), so it cannot be tested.

## Metrics (test set, parameters from the training set)

* RMSE on log cost: sqrt(mean((ln y_hat - ln y)^2)).
* MAPE: 100 x mean(|exp(ln y_hat) - y| / y), naive back-transform, no smearing correction.
* R^2 on log cost: 1 - SS_res / SS_tot, SS_tot about the test-set mean of ln y (may be
  negative).
* Paired bootstrap over test units: 2,000 resamples with replacement, `random.Random(0)`,
  percentile 95% intervals (sorted values at positions 50 and 1949) of
  dRMSE = RMSE_A - RMSE_B and dMAPE = MAPE_A - MAPE_B.
* Training-set metrics and fitted slopes are reported as well. The cost model suggests a
  slope near 1 for A; that is described, not tested.

## Decision rule

A predicts better if dRMSE < 0 and the interval's upper end is below 0; B predicts better
if dRMSE > 0 and the lower end is above 0; otherwise no detectable difference. The outcome
is reported whichever way it falls, with all metrics for A, B, S1 and S2.

## Secondary analyses

* Target total input tokens I (unweighted) instead of y, same models and split.
* Test metrics for the main-loop and the subagent test units separately, using the same
  fitted models (descriptive; the main-loop test subset is small).

## Limitations stated in advance

One operator, one harness, one provider; list prices are a weighting, not the operator's
bill (the operator is on a subscription); features and target come from the same
transcript, so the test is about the form of the relation, not about forecasting a
conversation before it runs.

## Output

Only aggregates and provenance go to `evidence/heldout.json`; per-unit rows stay in
`evidence/raw/` (gitignored). Every figure in the paper comes from that file via
`tools/gen_macros.py`.
