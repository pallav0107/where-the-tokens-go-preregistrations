# Pre-registration v2: head-to-head of context policies and a prefix-budget controller

Status: **PRE-REGISTERED**. The commit that first adds this file is the registration
timestamp; no v2 arm has run before it (no Gemini call of any kind has run in this project
when it was written, see DECISIONS.md D25). It extends, and does not replace,
`experiments/PREREG.md` (v1, with Amendment 1): same agent code base, tools, dataset,
retrieval, decoding and logging, unless stated here. Deviations go to DECISIONS.md.

## 1. Goal

Compare, on the same model, tools, tasks and turn cap, the cost and accuracy of published
context policies (re-implemented) against a new **prefix-budget controller** derived from the
cost model this paper tests (input cost = sum over requests of the re-sent prefix). Report the
cost-accuracy frontier for every arm, including arms where the controller loses. No claim
beyond this benchmark under these conditions.

## 2. Fixed conditions (all arms)

- Model: **G-Lite** (Amendment 1, A1 rule: highest `gemini-<v>-flash-lite`), Gemini API
  free tier, temperature 0, seed 42, thinking setting per A1. One model only, because the
  free-tier quotas are per project and the per-model limits are visible only in the provider's
  web console; daily limits reset at midnight Pacific.
- Tools: `search(query)` (BM25 over the v1 knowledge base, k set by the arm), `read(doc_id)`,
  `finish(answer)`; same system prompt text except for the arm-specific fields listed below.
- Turn cap 10 agent steps (helper requests such as a summariser are counted in cost and in
  requests, not in the turn cap). Same JSON action protocol as v1.
- Cost metrics per run: total input, cached input and output tokens (provider-reported), and
  list-price-equivalent USD at the cited G-Lite paid prices (DECISIONS.md D18; actual free-tier
  billing is zero). Accuracy: EM and F1 (official HotpotQA normalisation). Also turns,
  requests, latency.

## 3. Data split (frozen now)

- **Test** = the 48 v1 sample questions (PREREG.md section 4; positions 0-47 of
  `default_rng(20261115).permutation(7405)` over questions sorted by id). Touched once per
  final variant; never used for tuning.
- **Dev** = the next 24 questions of the same permutation that are not smoke questions:
  positions 54-77. Used for all tuning and debugging. Smoke questions (48-53) stay as in v1.
- Knowledge base for each split = pooled context paragraphs of that split's questions,
  built exactly as v1 section 4 (dev and test KBs are separate).

## 4. Arms

Baselines (each re-implemented from its source; the mapping is cited in claims.yaml):

| arm | context policy | k | LLM requests per step |
|---|---|---|---|
| `react` | carry everything (ReAct) | 2 and 8 | 1 |
| `notes_reset` | self-written notes + only the latest action/observation (v1 `reset_summary`; observation masking with a running summary) | 2 and 8 | 1 |
| `window2` | keep the last two action/observation pairs | 8 | 1 |
| `memgpt` | summarise-and-evict: when the history after S and Q exceeds 6,000 characters, the oldest half of the action/observation pairs is replaced by a summary written by a separate summariser request (same model), appended after Q | 8 | 1 (+ summariser when triggered) |
| `agentless` | no agent loop, fixed phases: retrieve for the question; one request writes a follow-up query; retrieve for it; one request answers from both passage sets | 8 | exactly 2 per run |
| `reflexion` | not run (budget); stated as such | - | - |

**Controller (`pbc`, the differentiator).** Every step the model returns, besides
thought/notes/action/args, a `cites` list of chunk ids it relies on. The controller then:
1. *Keep / summarise / drop per observation*: chunks of the last observation that are cited
   are appended verbatim to an evidence block E (keep); everything else in the observation is
   dropped after one viewing, its content surviving only in the model's notes (summarise or
   drop). Older actions and observations are never re-sent.
2. *Evidence retention*: a chunk cited by the current answer draft is never evicted. E is
   capped at `e_max` characters; above the cap the oldest chunks not cited in the current
   step are evicted.
3. *Cache-aware layout*: every request is `[S][Q][E][notes][last action][last observation]`;
   S, Q and E only ever grow by appending, so the longest possible prefix is identical
   between consecutive requests and eligible for the provider's implicit cache.
4. *Retrieval width*: searches use `k0`; if every chunk of the previous search went uncited
   (a miss), the next search uses `k_hi`, then returns to `k0`.
5. *Grounded answer*: `finish` must cite at least one chunk id actually retrieved in this run.
   An uncited answer is rejected once with a fixed message asking for supporting passages or
   another search; the second `finish` is accepted and flagged `grounded=false`.
Derivation from the cost model: a chunk carried for R more requests costs its tokens R times;
an uncited chunk has no evidence value, so it is carried once; a cited chunk is kept because
re-retrieving it costs more; small k minimises the per-step increment and is widened only
after a miss.

Tunable controller parameters (dev only): `k0` in {2, 3, 4}, `k_hi` in {6, 8}, `e_max` in
{3000, 6000} characters. Start values: k0 = 3, k_hi = 8, e_max = 6000.

## 5. Procedure

1. Dev round 0: every baseline arm/k above and `pbc` at its start values, on dev.
2. Up to two further dev rounds, each changing the controller in one place (a parameter or a
   bug fix), logged in DECISIONS.md with the dev table that motivated it. Baselines are not
   tuned (their k values are fixed above).
3. Freeze the final `pbc` (commit SHA recorded), then run every arm once on test.
4. Order inside a round: question-major, arms in a seeded shuffle (seed 20261117).
5. Quota: if the free-tier quota stops a round, it resumes after the daily reset from the
   artifact chain; arms never run are reported NOT RUN. Budget guard as in Amendment 1
   (list-price equivalent, `MAX_USD` 5 across everything).
6. Shared runs with v1: the test runs of `react` and `notes_reset` at k = 2 and 8 are, by
   construction, the v1 G-Lite cells (same prompt, code path, questions and settings), so they
   are run once and used by both analyses. v1's G-Flash cells run only after the v2 test, if
   quota and budget remain. Planned volume, at about 5 requests per looped run: dev round 0
   about 890 requests, later dev rounds about 100 each, test about 1,800.

## 6. Hypothesis H7 and statistics

(Named H7 because H6 is the existing-agents hypothesis of v1 section 7b.)

Let B\* be the baseline operating point (arm and k) with the highest test F1 (ties: lower
cost). Cost = mean list-price-equivalent USD per question. Paired cluster bootstrap over test
questions, 2,000 replicates, seed 7.

**H7 supported** iff, on test: (a) cost(pbc) / cost(B\*) <= 0.70 as a point estimate AND its
95% CI lies entirely below 1.0; (b) F1(pbc) - F1(B\*) >= -2.0 points as a point estimate; and
(c) no baseline operating point has both lower mean cost and higher mean F1 than `pbc`
(pbc is on the empirical frontier). The 95% CI of the F1 difference is reported; with 48
questions it is wide, and the paper says so. Otherwise H7 is not supported, reported as is.

Secondary (descriptive): the full cost-F1 table and frontier for every arm on test; cached
share per arm; total input tokens; grounded-answer rate; the same table on dev for each round.

## 7. Outputs

Raw JSONL in Actions artifacts; `experiments/analyze_v2.py` writes `evidence/agent_h2h.json`
(envelope schema, figures prefixed `Hth`), from which macros are generated. Every arm is
reported, including those where the controller is dominated.

## Amendment 2 (2026-11-15, before any v2 call; provider and model only)

Registered by the commit that adds this section. When it was written no OpenAI request of
any kind (smoke included) had been made by this project, no v2 arm had run, and no Gemini
call had ever run (D25, D27), so no data exists under the old provider. Where it conflicts
with sections 1-7, this amendment governs. **Everything else is unchanged**: arms and their k
values, the controller, its tunable grid and start values, tasks and the dev/test split,
turn cap 10, seed 42, temperature 0 (subject to B3), cell-order seed, procedure, H7 and its
statistics, and the outputs.

Reason: no Gemini API key exists or is coming (D13, D19, D25); the author directed that an
OpenAI API key be used (relayed by the coordinator, 15 Nov).

B1. **Model.** OpenAI API, model `gpt-6-luna`, Chat Completions (`POST /v1/chat/completions`),
replacing G-Lite in every arm. One model only, as before. The model page
(https://developers.openai.com/api/docs/models/gpt-6-luna, read 2026-11-15) lists a single
snapshot, `gpt-6-luna`, and the models list for this key shows no dated snapshot, so the
`model` field of every response is recorded per request (`response_model`).

B2. **Prices and USD.** Standard tier, short context (at most 272K input tokens), USD per 1M
tokens, from https://developers.openai.com/api/docs/pricing read 2026-11-15: input 0.10,
cached input 0.01, cache write (30-minute lifetime, the default) 0.125, output 0.50. The
prompt-caching guide (https://developers.openai.com/api/docs/guides/prompt-caching, read
2026-11-15) states that for GPT-5.6 and later models cache writes cost 1.25x the uncached
input rate, that implicit caching is billed at the write rate, that cache-write pricing is
not an additive fee, and that `cached_tokens` counts billed reads only. Per request:
USD = [(prompt - cached - cache_write) x 0.10 + cached x 0.01 + cache_write x 0.125 +
completion x 0.50] / 1e6, with the token counts of B5. This is the amount actually billed
(paid key); it replaces the "list-price-equivalent" USD of section 2 and D25, and H7's cost
is this USD. Requests send `service_tier: "default"` (standard pricing); a response that
reports any other tier is unpriced, which stops the block (D22 rule). Regional (data
residency) endpoints, which carry a 10% uplift, are not used.

B3. **Decoding and reasoning (rule fixed now, applied mechanically).** Every request sends
temperature 0, seed 42, `reasoning_effort` "none" (the lowest value on the model page; the
default is "medium"), JSON mode (`response_format: {"type": "json_object"}`, as for the other
providers), `max_completion_tokens` 4096 (reasoning tokens included) and `store: false`. If the
API rejects a parameter with HTTP 400 naming it, the request is re-sent without it
(temperature, seed: the model default applies) or, for `reasoning_effort`, with the next
lowest value of none -> minimal -> low. Rejected requests are not billed and are not data.
The change is sticky for the provider instance; one instance serves every arm of a block, so
all arms use identical parameters, and every request records the parameters actually sent.
The smoke call (B6) settles this before any data, and its outcome is recorded in
DECISIONS.md. A fallback is a reported property of the model, not a deviation.

B4. **Prompt caching.** Implicit caching only, as the provider applies it by default. No
`prompt_cache_key`, `prompt_cache_options`, `prompt_cache_retention` or explicit cache
breakpoint is sent, for any arm (identically none), so the arms differ only in their context
policy. Minimum cacheable prompt: 1,024 tokens (GPT-5.6 and later, per the guide).

B5. **Metric mapping** (existing metrics, definitions unchanged). Input tokens =
`usage.prompt_tokens` (includes cached and cache-written tokens); cached input =
`usage.prompt_tokens_details.cached_tokens` (billed cache reads; absent = 0); output =
`usage.completion_tokens`, which includes the reasoning tokens (billed as output). Recorded
per request, descriptive only and outside every decision rule:
`completion_tokens_details.reasoning_tokens`, `prompt_tokens_details.cache_write_tokens`, the
full `usage` object, `system_fingerprint`, `service_tier` and `finish_reason`. Cached share =
cached / input, as before. Reasoning and cache-write tokens are also reported per arm.

B6. **Smoke call.** One request before any data, on synthetic text (the v1 system prompt with
the invented fixture `agent/tests/fixtures/hotpot_tiny.jsonl`; no HotpotQA question of any
split), labelled `smoke` and never used as data. It settles B3 and shows the usage fields.
Its cost counts against the budget and is recorded in the committed off-chain spend ledger
`experiments/spend/spend_ledger.jsonl`, which every CI job adds to its prior spend.

B7. **Budget and the [REDACTED: unrelated].** `MAX_USD` = 5 in total for everything this project
spends (the figure is unchanged; spend so far is zero, as no cloud call has run). On top of
the block estimate (A4, priced at the highest input rate, 0.125) and the hard stop, every
request is admitted only if spend so far plus its worst case is at most `MAX_USD`. The worst
case is an upper bound on prompt tokens (UTF-8 bytes of all messages plus a per-message and a
fixed allowance) at 0.125, plus `max_completion_tokens` at 0.50. A request whose bound exceeds
272K tokens is refused (long-context prices differ). Each response's actual USD is added at
once, and the running total (`cum_usd`) is recorded per request. [REDACTED: unrelated]
[REDACTED: unrelated], so: at most 2 concurrent requests (the grid is
sequential, so 1); HTTP 429 rate limits are retried with exponential backoff (2, 4, 8, 16, 32
and 64 s, or the server's Retry-After capped at 120 s) and 5xx or network errors as in v1
(5/15/45 s), after which the block stops (NOT RUN: quota) or the run errors (section 9). An
`insufficient_quota` error is never retried: the whole run stops immediately, every
unfinished cell is NOT RUN (reason: insufficient_quota), the author is told, and no further
request is made until the author confirms.

B8. **Where runs execute.** Unchanged: dev and test rounds run as `experiment.yml` jobs on
the [REDACTED: infrastructure] runner (D16), with raw JSONL in Actions artifacts and spend carried through the
artifact chain (D22). The key comes from the Actions secret `OPENAI_API_KEY`
(workflow_dispatch only); a missing key means NOT RUN. Only the B6 smoke call runs from a
local workstation, with the key read at run time from the git-ignored `.env.local`.

B9. **Consequences.** (a) Section 2's free-tier wording, the daily-reset rule of section 5.5
and the Gemini data-use terms (Amendment 1, A8) concern Gemini and lapse. The content sent is
public HotpotQA text (CC BY-SA 4.0), the fixed prompts and the model outputs, with `store`
false. (b) Section 5.6: v1 (PREREG.md with Amendment 1) is **not** amended here, and its
Gemini cells stay NOT RUN (no key). The v2 test runs of `react` and `notes_reset` keep the v1
code path, prompt and settings, but they are gpt-6-luna runs and serve v2 only. (c) Planned
volume is unchanged: about 890 requests in dev round 0, about 100 per later dev round and
about 1,800 for test.
