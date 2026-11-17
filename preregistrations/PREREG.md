# Pre-registration: context carry-over and the cost of LLM agent runs

Status: **PRE-REGISTERED**. The git commit that first adds this file is the registration
timestamp. No experimental run (any question of the sample below, any cell) was executed
before that commit. Every later change to this file, and every deviation from it, is
logged with a reason in `experiments/DECISIONS.md`; this file is never edited silently.

Author: [REDACTED: personal]. Paper: AGENT 2027 workshop @ ICSE 2027 (short paper).

## 1. Question

Observational data from one production harness (its Claude+Kimi logs) suggest that the
input-token cost of an LLM agent run tracks (context size x number of requests), because
every request re-reads the growing prefix, rather than prompt size. This experiment tests
that claim on a fresh, minimal agent that anyone can rerun, with the context policy and the
retrieval width manipulated directly.

## 2. Agent under test (`agent/`)

- ReAct-style loop. One LLM request per step. Each step the model emits exactly one JSON
  object `{"thought", "notes", "action", "args"}`; `action` is one of
  `search` (args `query`; returns the top-k chunks, k fixed by the cell, never chosen by the
  model), `read` (args `doc_id`; returns the full paragraph) or `finish` (args `answer`).
- `notes` is a running summary of facts found so far. It is requested in **every** step
  under **every** policy, so the prompt text is identical across policies; only what is
  re-sent differs.
- Invalid JSON / unknown action: the observation is a fixed error message and the step
  counts. Turn cap: 10 requests. A run that hits the cap without `finish` gets answer `""`.
- Retrieval: BM25 (k1=1.5, b=0.75, lowercase alphanumeric tokens, no stemming, no
  stopword removal) over the chunked knowledge base (Section 4). Deterministic.
- Decoding: temperature 0, seed 42, `num_predict` 4096 (local), `num_ctx` 131072 (local).
  Reasoning setting per model is fixed in Section 5. Model "thinking" text is never
  re-sent in later requests under any policy.
- Thinking/visible output tokens are both counted in output tokens as the provider reports.

### Context policies (the manipulated factor)

Let request t carry the message list M_t. S = system prompt, Q = question, (A_i, O_i) =
the model's action message at step i and the resulting observation.

- `carry_all`: M_t = [S, Q, A_1, O_1, ..., A_{t-1}, O_{t-1}]. Every observation stays.
- `reset_summary` (MemGPT-style eviction with a self-written summary):
  M_1 = [S, Q]; M_t = [S, Q, A_{t-1}, O_{t-1}] for t >= 2. The latest action carries the
  model's own `notes` (its summary of everything earlier); all older actions and
  observations are dropped. Each observation is therefore sent exactly once.
- `sliding_window(n)`: implemented, **not** part of this experiment.

## 3. Factors and cells

| factor | levels |
|---|---|
| context policy | `carry_all`, `reset_summary` |
| retrieval top-k | 2, 8 |
| model (local, [REDACTED: infrastructure] Ollama) | `glm-4.7-flash`, `nemotron-3-nano:30b` |
| model (cloud, only if a key is present in the job environment), in priority order | (1) `qwen-flash` and (2) `qwen-plus` via DashScope International pay-as-you-go (OpenAI-compatible); (3) Gemini Flash tier via the Gemini API; (4) `qwen-max` only if the remaining budget allows after (1)-(3) |

Provider rules: Qwen cells use only the pay-as-you-go key and the `dashscope-intl`
compatible-mode endpoint; the adapter refuses Alibaba Coding Plan keys/endpoints (their
terms exclude automated scripts and batch calls). No Gemini API key exists or is expected
(the available Gemini access is CLI-only), so the Gemini-API cells are expected to be
recorded NOT RUN (reason: no API key); Gemini is instead measured in the existing-agents arm
(Section 7b).

Gemini model resolution (fixed now, applied at run time because no key exists yet): list
the models visible to the key (`GET v1beta/models`); keep ids of the exact form
`gemini-<version>-flash` that support `generateContent` (no `-lite`, `-preview`, `-exp`,
`-image`, `-tts`, `-live` or date suffix); take the highest version. The exact id is
recorded in every request record and in the evidence.

Local grid: 2 x 2 x 2 = 8 cells x N questions. Cloud cells use the same 4 policy x k cells
and the same questions, run in the priority order above. A cloud cell whose key is absent
is recorded as **NOT RUN** (with the reason), never silently skipped. Budget: `MAX_USD` = 5
USD **in total across all cloud runs** unless the author raises it. Each cloud model is run
as one block (its 4 cells x N questions). Before a block starts, (actual spend so far +
estimated cost of the whole block, priced at the full uncached input rate with a per-run
token estimate taken from the completed local runs and logged in DECISIONS.md) must not
exceed `MAX_USD`, or the block is refused and its cells recorded as NOT RUN (budget); a
running block stops when actual spend reaches `MAX_USD`, and its incomplete cells are
reported as such. Actual spend is computed from the provider
usage fields and a dated, cited price sheet (`agent/prices.yaml`; URLs and retrieval dates
also in `experiments/DECISIONS.md`).

Model contingency (decided now, applied mechanically): before the main grid, each local
model gets a smoke test of 6 questions x 1 cell (`carry_all`, k=8) on the smoke-test
questions (Section 4, disjoint from the sample). A model is replaced if >= 3 of the 6 smoke
runs end with a provider error or with a JSON-parse error on >= half of their steps.
The replacement is a Qwen3 long-context local model. (`gpt-oss:20b` was excluded before registration: it answers
with native tool calls and empty message content instead of the shared JSON protocol; see
DECISIONS.md D11.)
Smoke-test runs are excluded from every analysis. Models whose name contains `ablit` are
never used.

Run order: all cells of one question are run before the next question; questions are run in
the sampled order (Section 4); cell order within a question is a fixed seeded shuffle
(seed 20261116). Requests are sequential (one in flight).

## 4. Task set

- HotpotQA, distractor setting, validation split (7,405 questions), licence CC BY-SA 4.0,
  file `distractor/validation-00000-of-00001.parquet` from the Hugging Face dataset
  `hotpotqa/hotpot_qa`; its SHA-256 is recorded in every run's provenance.
- Sample: sort all questions by `id`; draw indices with
  `numpy.random.default_rng(20261115).permutation(7405)`. The first **N = 48** indices are
  the sample, in that order (this is also the run order). Indices 49-54 of the same
  permutation are the smoke-test questions.
- Knowledge base: the union of the context paragraphs (10 per question: 2 supporting + 8
  distractors) of the 48 sampled questions, de-duplicated by title (first occurrence kept).
  Each paragraph is one document (`doc_id` = stable hash of the title). Chunks are
  consecutive windows of up to 3 sentences within a paragraph (no overlap), prefixed with
  the title.
- Held-out split for H1 (by question, so no question appears in both): sampled position
  p = 0..47; test iff p mod 3 == 2 (16 questions), train otherwise (32 questions).

## 5. Fixed per-model settings

| model | provider | reasoning | notes |
|---|---|---|---|
| glm-4.7-flash | ollama | `think: false` | `format: json` |
| nemotron-3-nano:30b | ollama | `think: false` | `format: json` |
| a Qwen3 long-context local model (contingency) | ollama | `think: false` | `format: json` |
| Gemini Flash (resolved id) | gemini | thinking budget 0 where the model allows it, else the minimum allowed | JSON mime type |
| qwen-flash | dashscope | `enable_thinking: false` | `response_format: json_object` |
| qwen-plus | dashscope | `enable_thinking: false` | `response_format: json_object` |
| qwen-max (budget permitting) | dashscope | (no thinking mode) | `response_format: json_object` |

Model digests (Ollama `/api/tags`) and the server version are recorded per run.

## 6. Measurements

Per request (JSONL, one line per LLM request): run id, cell, question id, step, provider-
reported input tokens `x` (Ollama `prompt_eval_count`; Gemini `promptTokenCount`;
DashScope `prompt_tokens`), provider-reported cached input tokens (Gemini
`cachedContentTokenCount`, absent field = 0; DashScope `prompt_tokens_details.cached_tokens`,
absent = 0; Ollama does not report it, so it is recorded as `null`, never as 0), output
tokens (including reasoning tokens where reported separately), wall-clock latency
and provider timing fields, the action taken, observation length (chars), and the request
cost in USD for cloud providers.

Per run: total input Y = sum_t x_t; total cached; total output; number of requests T;
initial prompt size P = x_1 (system prompt + tool descriptions + question); latency;
finished flag; answer; exact match (EM) and token F1 against the gold answer using the
official HotpotQA normalisation; USD (cloud).

Token-ledger view: each run is written as one tokenledger "turn", each request as a
"step", with `priming_tokens = P` and `conversation_tokens = x_t - P`. The cost structure
is computed with `tokenledger` (PyPI `tokenledger-llm`).

## 7. Hypotheses, statistics and decision rules

All confidence intervals are 95% percentile intervals from a **cluster bootstrap that
resamples questions** (2,000 replicates, seed 7). "Supported" / "not supported" is decided
only by the rules below; anything else is descriptive.

Scope of the verdicts: H1-H4 are decided on the **local** cells (the two [REDACTED: infrastructure] models).
Cloud models that complete their block are analysed with the same code and reported
separately per model; they never change the primary verdicts.

**H1 (context x turns predicts cost; prompt size does not).** Fit on the train split, score
on the test split, separately per model, then pooled over models.
- M_ctx (primary): Y_hat = Cbar_cell x T, where Cbar_cell = (sum of x) / (sum of T) over
  the training runs of the same cell (model x policy x k). No other fitted parameter.
- M_prompt (primary baseline): OLS Y ~ 1 + P on the training runs.
- Secondary comparators (reported, not decisive): M_cell (training mean of Y for the
  cell, no turns); M_carry (OLS Y ~ P*T + G_policy(T) with one coefficient per k, where
  G_carry(T) = T(T-1)/2 and G_reset(T) = T-1).
- Metrics on test runs: R^2 (1 - SSE/SST around the test mean), MAE, MAPE.
- Supported iff pooled R^2(M_ctx) >= 0.80 AND the CI of MAPE(M_prompt) - MAPE(M_ctx)
  lies entirely above 0.

**H2 (superlinear under carry_all, ~linear under reset_summary).** Step 1 carries no
observation under either policy, so the exponent is fitted from step 2 on. For each run
with T >= 3, let C_r(n) = sum_{s=2}^{n+1} x_{r,s} for n = 1..T-1. Fit
log C_r(n) = a_r + b log n with run fixed effects, separately per policy, pooled over models
and k (per model x k also reported).
- Supported iff (a) CI(b_carry) lies entirely above 1, (b) CI(b_carry - b_reset) lies
  entirely above 0, and (c) the point estimate of b_reset is within [0.90, 1.10].
- Partially supported iff (a) and (b) hold but (c) does not.
- Secondary (descriptive): across-run fit log Y ~ log T per policy.

**H3 (retrieved chunks are re-carried: top-k x policy interaction).** For each question q
and model m: I_qm = (Y[carry,8] - Y[carry,2]) - (Y[reset,8] - Y[reset,2]), and
rho = mean(Y[carry,8] - Y[carry,2]) / mean(Y[reset,8] - Y[reset,2]). Under
`reset_summary` each retrieved chunk is sent exactly once, so the denominator is the k-effect
when retrieved tokens are not re-carried.
- Supported iff CI(mean I) lies entirely above 0 AND CI(rho) lies entirely above 1. If the
  denominator's CI includes 0, rho is reported as undefined and H3 is decided on I alone,
  stated as such. Mean T per cell is reported alongside (k can change T).

**H4 (accuracy).** EM and F1 per cell with CIs, and paired differences between policies and
between k. No direction is hypothesised and no claim of improvement or harm is made.

**H5 (cloud only).** (a) Cached share s_t = cached_t / x_t rises with step: run-fixed-effect
OLS slope of s on t (t >= 2) has a CI entirely above 0. (b) USD diverges from token volume:
the OLS slope of (USD_run / Y_run) on T has a CI entirely below 0. Reported only for cloud
cells that ran; otherwise "not run".

## 7b. Existing-agents arm (stretch priority; run only after the core arm is complete)

Third-party agents are run **unchanged** through their official non-interactive modes and
measured only from their own reported usage. They run on a local workstation (where
their logins live), sequentially, respecting rate limits.

- Agents: Gemini CLI headless (`gemini -p <prompt> -o json`, default approval mode, so only
  tools that need no confirmation run; version recorded; tokens from its JSON `stats` per model: prompt, candidates, cached, thoughts,
  tool, total, and request counts); Kimi Code headless via the production harness
  (`[REDACTED: infrastructure] run`, read-only plan mode; requests and input tokens from
  `[REDACTED: infrastructure] report`, which reads Kimi's own store). Gemini CLI signed in with OAuth
  receives no context caching, so its cached = 0 is a property of the auth path, not a
  finding. The arm runs only if the CLI authenticates headlessly with an existing
  login; otherwise it is recorded NOT RUN with the reason.
- Tasks: the first 16 sampled questions (positions 0-15). Each task gets a fresh working
  directory holding the knowledge base as one text file per document (`kb/<doc_id>.txt`,
  content "Title: paragraph"), and the fixed prompt: "Answer the question using only the
  files in ./kb. Reply with only the short answer (a name, date, number, or yes/no).
  Question: <question>". No other instruction, tool restriction or context policy is
  imposed; the agents' own defaults are what is measured.
- Outcomes per task: total prompt (input) tokens, cached tokens, output tokens, number of
  model requests, wall-clock time, EM/F1 of the reply.
- **H6.** Total prompt tokens per task grow superlinearly with the number of model
  requests, as context x turns predicts. Test per agent: OLS of log(total prompt tokens) on
  log(requests) across tasks; supported iff the 95% bootstrap CI (resampling tasks, 2,000
  replicates, seed 7) of the slope lies entirely above 1. With 16 tasks this test may be
  inconclusive; that is reported as such. This is a cross-task, observational comparison
  within each agent (task difficulty varies with requests), stated as a limitation.
- If an agent's quota or rate limit stops the arm, it is stopped cleanly and the remaining
  tasks are recorded NOT RUN (reason: quota). No credentials are extracted or reused outside
  the agents' own tools.

## 8. Fail-first controls (each must be shown to fail on broken input)

1. **Token accounting.** A 3-step scripted run (mock provider whose input count is the number
   of words in the request, so each prefix sum is computed by hand from the scripted texts:
   47 under `carry_all`, 41 under `reset_summary`) must reproduce the hand-computed sums
   exactly, through the agent's logger and through tokenledger; the test is shown to fail
   when the logger drops a request (`agent/tests/test_contract.py`).
2. **Broken policy.** 16 runs (first 16 sampled questions, `glm-4.7-flash`, k=8) of a
   deliberately broken `reset_summary` that writes notes but does not drop anything. The H2
   analysis, given these runs in place of the real reset runs of the same questions, must
   NOT return "supported" (it must fail rule (b) or (c)). These runs are excluded from the
   hypothesis analyses.
3. **Label shuffle.** Permute Y across runs within each model (200 permutations, seed 11)
   and rerun the H1 decision rule. The control passes iff the rule returns "supported" in
   at most 5% of permutations.

## 9. Exclusions and missing data

- Excluded (and counted per cell): runs with a provider/infrastructure error that persists
  after 3 retries; runs where any request's input reaches `num_ctx` (possible truncation).
  An excluded run is retried once at the end of the grid; if it fails again it stays
  excluded.
- Not excluded: parse errors, turn-cap runs, wrong answers (they are agent behaviour and
  real cost).
- Deadline rule: if the full local grid is not complete by 2026-11-18 12:59 UTC, the
  analysis uses complete question blocks only (all 8 local cells of a question), in sampled
  order, and the number analysed is reported as a deviation.

## 10. Outputs and provenance

- Raw request JSONL and run summaries are uploaded as GitHub Actions artifacts of the
  `experiment.yml` run that produced them (private repository); they are not committed.
- `experiments/analyze.py` writes `evidence/agent_runs.json`: every number in the paper's
  controlled-experiment section, plus provenance (Actions run URLs and ids, git SHA,
  dataset SHA-256, model digests, Ollama version, PREREG commit SHA, tokenledger version).
  LaTeX macros are generated from it by the pipeline's `gen_macros.py`.
- Hypotheses not yet evaluated are reported as "not yet run". Negative and partial results
  are reported as they are.

## Amendment 1 (2026-11-15, before any run; reason: author instruction "Gemini only")

Registered by the commit that adds this section. No question of the sample had been run
when it was written. Where it conflicts with sections 1-10 above, this amendment governs;
everything not mentioned here is unchanged.

A1. **Models.** The [REDACTED: infrastructure] is not used at all. All Ollama and DashScope (Qwen) cells are
removed. The custom-agent arm uses two Gemini API models of different price tiers, selected
at run time from the models visible to the key (`GET v1beta/models`, `generateContent`
supported, no `-preview`, `-exp`, `-image`, `-tts`, `-live`, `-customtools` or date suffix):
- **G-Flash**: the highest version of the exact form `gemini-<version>-flash`;
- **G-Lite**: the highest version of the exact form `gemini-<version>-flash-lite`.
Pro-tier models are excluded: the only current one is a preview, and its price does not fit
the budget for a full block. Exact model ids are recorded per request. Reasoning: thinking
budget 0 where the model accepts it; for `gemini-3*` ids, `thinkingLevel` "minimal", else
"low"; the setting used is recorded. Temperature 0 and seed 42 stay as registered.
Per request, the full `usageMetadata` is recorded (`promptTokenCount`,
`cachedContentTokenCount` (absent = 0), `candidatesTokenCount`, `thoughtsTokenCount`).

A2. **Grid.** Policy {carry_all, reset_summary} x k {2, 8} x model {G-Flash, G-Lite} =
8 cells x 48 questions, sampled exactly as in section 4. Run order as in section 3.

A3. **Smoke test and contingency** (replaces the Ollama contingency): 6 smoke questions x
(carry_all, k=8) per model, same failure rule as section 3. A failing model is replaced by
the next-highest version of the same tier form. Smoke runs are excluded from analyses but
their cost counts toward the budget.

A4. **Budget.** `MAX_USD` = 5 USD in total for everything that calls the Gemini API in this
project: literature embeddings, smoke runs, the broken-policy control and both model
blocks. Order: embeddings, smoke tests, broken-policy control (16 runs, G-Lite, k=8), the
G-Flash block, then the G-Lite block. A block's pre-start estimate is the mean input and
output tokens per run of that model's smoke runs (carry_all, k=8, the costliest cell) times
its number of runs, priced at the full uncached input rate; the rest of the budget rule in
section 3 is unchanged. If the provider's quota or rate limit is exhausted (HTTP 429 that
persists after backoff, or a per-day quota error), the block stops cleanly and its
unfinished cells are recorded NOT RUN (reason: quota).

A5. **Verdict scope.** H1-H4 are decided on the Gemini custom-agent cells (both models
pooled, each model also reported). H5 is decided on the same cells, which are now the
primary cells rather than optional cloud cells. The broken-policy control (section 8.2) uses
G-Lite at k=8.

A6. **Literature RAG embeddings** use the Gemini API: the highest-version
`gemini-embedding-<n>` model supporting `embedContent`, task types RETRIEVAL_DOCUMENT and
RETRIEVAL_QUERY. Until the key exists, retrieval for the literature RAG is BM25 and is
labelled as such. (The claims gate does not depend on the retriever: every quote is
checked verbatim against its cached chunk.)

A7. **Existing-agents arm (7b).** Only Gemini CLI headless is run (the Kimi Code arm is
dropped). H6 and the rest of section 7b are unchanged.

A8. **Data-use terms.** If the key's Google Cloud project has no billing, the Gemini API
Free-tier terms apply: submitted content and responses may be used to improve Google
products and may be read by human reviewers. The content sent is public HotpotQA text
(CC BY-SA 4.0), the fixed prompts and model outputs; no personal or confidential data.
Whether the key is on the free or paid tier is recorded in DECISIONS.md when it arrives.
