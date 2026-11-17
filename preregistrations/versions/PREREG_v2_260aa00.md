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
