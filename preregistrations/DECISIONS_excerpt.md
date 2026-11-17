# Decision log: controlled experiment + literature RAG

Append-only. Each entry: date (UTC calendar date in this release), decision, reason, and whether it deviates from
`experiments/PREREG.md`. Deviations after the PREREG commit are marked **DEVIATION**.

## Up to 2026-11-15 11:42 UTC (before the PREREG commit; no run of any sampled question has happened)

[NOT INCLUDED: D1 is not referenced by the released pre-registrations]



[NOT INCLUDED: D2 is not referenced by the released pre-registrations]






[NOT INCLUDED: D3 is not referenced by the released pre-registrations]



[NOT INCLUDED: D4 is not referenced by the released pre-registrations]




[NOT INCLUDED: D5 is not referenced by the released pre-registrations]


[NOT INCLUDED: D6 is not referenced by the released pre-registrations]



[NOT INCLUDED: D7 is not referenced by the released pre-registrations]


[NOT INCLUDED: D8 is not referenced by the released pre-registrations]






[NOT INCLUDED: D9 is not referenced by the released pre-registrations]




[NOT INCLUDED: D10 is not referenced by the released pre-registrations]














- **D11 Local models: gpt-oss:20b excluded, nemotron-3-nano:30b in.** Engineering check on the
  4-question synthetic fixture (`agent/tests/fixtures/hotpot_tiny.jsonl`, invented text, not
  HotpotQA; output kept [REDACTED: infrastructure]): `gpt-oss:20b` had
  parse errors on 33 of 48 steps (all 30 under reset_summary). Inspection showed Ollama
  returning empty `content` with a native harmony `tool_calls` entry (e.g. `search` with
  `{"query": ...}`), with or without `format: json`, and also with a reworded prompt that
  says functions cannot be called. A tool call carries no `notes`, so the model cannot follow
  the shared text protocol that the policy manipulation depends on. `glm-4.7-flash`,
  `nemotron-3-nano:30b` and a Qwen3 long-context local model each returned valid JSON actions on the same
  probe. Registered models: glm-4.7-flash and nemotron-3-nano:30b; contingency
  a Qwen3 long-context local model. Broken-policy control model changed to glm-4.7-flash accordingly.
[NOT INCLUDED: D12 is not referenced by the released pre-registrations]


- **D13 Provider rules after a licensing check (coordinator, by 15 Nov).** Qwen uses
  only the pay-as-you-go key on the `dashscope-intl` compatible-mode endpoint; the adapter
  refuses Coding Plan keys (`sk-sp-`) and `coding*` endpoints, because the Coding Plan terms
  (https://help.aliyun.com/en/model-studio/coding-plan) exclude automated scripts and batch
  calls. No Gemini API key exists or is coming; Gemini-API cells will be NOT RUN (no API
  key; the available access is CLI-only). The coordinator confirmed `DASHSCOPE_API_KEY` set as
  an Actions secret on 2026-11-15 11:11 UTC.
[NOT INCLUDED: D14 is not referenced by the released pre-registrations]





## 2026-11-15 (after the PREREG commit 1f63cd0, 11:42 UTC; still before any run)

- **D15 PREREG registered.** Commit `1f63cd0c11963fff8b98422135b174d79f39d677`
  (2026-11-15T11:42:00Z), pushed to `exp/agent-rag` on GitHub the same minute, on top of
  `origin/main` cc4d072. It froze PREREG.md, this log, the agent, the grid config, the
  contract tests and the workflow; analyze.py lands in a later commit, implementing the
  decision rules already fixed in PREREG section 7.
- **D16 How jobs run on the [REDACTED: infrastructure] (implementation, not a design change).** The repository's
  only runner ([REDACTED: infrastructure]) is the pipeline's hardened container: it has no pip
  and serves CI. A dedicated second runner was considered and NOT deployed (the agent
  tooling declined the action). Instead every experiment job runs on the [REDACTED: infrastructure] runner but
  executes inside a throw-away, digest-pinned `python:3.12-slim` container
  (`experiments/ci/run_in_container.sh`, `docker create` + `docker cp`, the same pattern as
  the pipeline's PDF build), with host networking to reach the [REDACTED: infrastructure], pinned Python deps
  (`experiments/ci/requirements-pinned.txt`) and host cache directories for the dataset,
  the literature index and pip. Grid jobs are dispatched in steps of about an hour
  (`questions` input, resuming from the previous artifact) so CI is not blocked for long.
  Each step's Actions log and artifact is provenance; the resume chain is recorded in the
  evidence file.
- **D17 Author: Gemini only (pre-run deviation, registered as PREREG Amendment 1).** The
  coordinator relayed the author's instruction, superseding D8, D11 and D13: do not use the
  [REDACTED: infrastructure] at all; Gemini API models only (key `GEMINI_API_KEY`, being created in the provider console,
  not yet set); literature embeddings via Gemini; existing-agents arm = Gemini CLI only.
  Reason recorded as given: "author: Gemini only". No question of the sample had been run.
  Consequences: the Ollama and Qwen adapters stay in the code (tested, unused); the grid
  config lists only the two Gemini tiers; the [REDACTED: infrastructure] was used before this instruction only
  for the D2 accounting probe, the D11 engineering check on synthetic text, and a literature
  embedding pass that was killed before its single database commit; that index was deleted
  and rebuilt without vectors (35 sources, 3,434 chunks, 0 vectors), so no [REDACTED: infrastructure] embedding
  is used anywhere.
- **D18 Gemini price sheet and terms (retrieved 2026-11-15).**
  https://aistudio.google.com/docs/pricing (redirect target of
  https://ai.google.dev/gemini-api/docs/pricing), paid tier, USD per 1M tokens:
  gemini-3.8-flash and gemini-3.6-flash input 0.75, cached 0.075, output 3.75 through
  2026-12-31 (1.50 / 0.15 / 7.50 from 2027-01-01); gemini-3.5-flash-lite 0.30 / 0.03 / 2.50;
  gemini-3.1-flash-lite 0.25 / 0.025 / 1.50; gemini-embedding-2 0.20 (text input),
  gemini-embedding-001 0.15. Pro: gemini-3.1-pro-preview 2.00 input / 12.00 output (preview,
  excluded). The page marks the Free tier as "Content used to improve our products" and the
  paid tier as "Content not used to improve our products". Gemini API Additional Terms
  (https://aistudio.google.com/docs/terms, effective 2026-03-23): for Unpaid Services
  "human reviewers may read, annotate, and process your API input and output"; for Paid
  Services "Google doesn't use your prompts ... or responses to improve our products".
  Whether the key is billed (paid terms) or free is to be recorded when it arrives.
- **D19 Gemini CLI headless auth.** On the local workstation, [REDACTED: infrastructure] selects auth type
  `gemini-api-key` and no key is set, so `gemini -p ... -o json` exits with code 41
  ("When using Gemini API, you must specify the GEMINI_API_KEY environment variable").
  Setting `GOOGLE_GENAI_USE_GCA=true` for the process did not override it. The local CLI
  configuration was not changed. The Gemini CLI arm stays NOT RUN until the author switches
  the CLI to "Login with Google" (or says which auth it should use).
[NOT INCLUDED: D20 is not referenced by the released pre-registrations]












[NOT INCLUDED: D21 is not referenced by the released pre-registrations]








- **D22 Spend guard hardened before any Gemini call (independent review, 15 Nov).** A
  reviewer found ways the budget could be exceeded or under-counted: the end-of-grid retry
  had no guard; spend of failed/partial attempts was not counted; spend was carried across
  jobs only by a hand-typed input; the broken-policy control was not restricted to G-Lite;
  the model was re-resolved per run (an unpriced id would have counted as 0 USD). All fixed,
  each with a test shown to fail on the old code: prior spend is now summed automatically
  from every `requests*.jsonl` and embedding log carried forward in the artifact chain
  (`--spend-from out`), every Gemini-calling job refuses to start without the previous
  artifact unless it is explicitly first in the chain, the provider is resolved once per
  block, unpriced spend stops the block, retries are budget-checked, and the full
  `usageMetadata` and thinking setting are now written per request (Amendment A1). The
  vectorised analysis was checked against the earlier loop version on synthetic data:
  identical estimates, CIs and decisions (max difference 1.6e-13).

## 2026-11-15 to 2026-11-16 (UTC)

[NOT INCLUDED: D23 is not referenced by the released pre-registrations]











[NOT INCLUDED: D24 is not referenced by the released pre-registrations]





- **D25 Free tier.** The author chose the Gemini free tier. The budget guard still prices every
  request at the paid list prices of D18, so `MAX_USD` acts as an upper bound on the
  list-price equivalent; actual billing on the free tier is zero. Free-tier terms apply
  (Amendment A8, D18: content may be used to improve Google products and read by human
  reviewers; only public HotpotQA text, fixed prompts and model outputs are sent). Free-tier
  rate limits may stop a block with a quota error; such cells are recorded NOT RUN (quota)
  and resumed on a later day from the artifact chain (A6 fix in D22).
[NOT INCLUDED: D26 is not referenced by the released pre-registrations]


- **D27 Author goal: efficient agent, fair head-to-head (coordinator, 15 Nov).** Registered as
  `experiments/PREREG_v2.md`, commit 260aa00 (2026-11-15T13:53:21Z), before any v2 arm
  or Gemini call. Free-tier per-model limits could not be read from public docs (they are
  shown per project at aistudio.google.com/rate-limit; the docs state limits are per project
  and RPD resets at midnight Pacific), so run counts were sized conservatively (G-Lite only;
  shared v1/v2 runs) and the quota-stop rule applies. Novelty check: 8 recent context-
  management papers added (context folding, ACON, AgentFold, MEM1, ACE, FoldAct, CTXAgent,
  PAACE; 4,276 chunks from 43 sources; 63 verified claims) and summarised in
  `literature/NOVELTY.md`. The related-work text states what is not new (compressing or
  dropping history; dollar cost reported by CTXAgent) and positions the controller only by
  its combination of features. No "best" or "state-of-the-art" wording.
[NOT INCLUDED: D28 is not referenced by the released pre-registrations]










- **D29 Provider change to OpenAI `gpt-6-luna` (PREREG_v2 Amendment 2; author via coordinator,
  15 Nov).** No Gemini key exists, so the author directed use of an OpenAI API key. Registered
  as PREREG_v2 Amendment 2 before any OpenAI request, smoke included. No v2 arm and no Gemini
  call had run, so no data exists under the old provider. Only the provider and model change.
  Arms, tasks, splits, turn cap, seeds, metrics and analysis are unchanged.
  - **Key handling.** The key is read at run time from the git-ignored `.env.local` (smoke
    call only) or from the Actions secret `OPENAI_API_KEY` (set by the author,
    2026-11-15T23:05Z UTC; workflow_dispatch only). It is never printed, logged, committed
    or placed in an error message; error bodies are scrubbed of `sk-` strings. [REDACTED: unrelated]
    [REDACTED: unrelated]
    [REDACTED: unrelated], hence the B7 rules: a per-request worst-case guard
    under `MAX_USD` = 5, at most 2 concurrent requests, bounded backoff on 429 and 5xx, and
    an immediate whole-run stop on `insufficient_quota`.
  - **Docs read 2026-11-15 (developers.openai.com).** The model page lists one snapshot
    (`gpt-6-luna`); a reasoning model; `reasoning.effort` none/low/medium (default)/high/
    xhigh/max; Chat Completions and Responses supported. It does not say whether `temperature`
    or `seed` are accepted, and neither does the Chat Completions reference (`seed` is "best
    effort", Beta) or the reasoning guide, hence the mechanical B3 rule. The pricing page,
    standard tier, short context: input 0.10, output 0.50, cached input 0.01, 30-minute cache
    write 0.125, 12-hour cache write 0.25. Its Fast tier (formerly Priority) is 2x, and
    regional endpoints carry a +10% uplift. Hence `service_tier: "default"`, and any other
    reported tier is unpriced. The cache-write rate was not in the coordinator's price list;
    the prompt-caching guide confirms that implicit caching on GPT-5.6 and later bills writes
    at 1.25x input and reports them as `prompt_tokens_details.cache_write_tokens`
    (`cached_tokens` = billed reads only). The cost formula therefore prices writes
    separately (Amendment 2, B2) and the worst-case guard uses the write rate. The minimum
    cacheable prompt is 1,024 tokens.
  - **Cache routing.** No `prompt_cache_key` or other cache option is sent, for any arm.
  - **Implementation.** `OpenAIProvider` (stdlib HTTPS, like the other adapters);
    `agent.budget.SpendGuard`, attached by the grid runner to each OpenAI block with the
    prior spend, without which the provider refuses to send; the off-chain spend ledger
    `experiments/spend/spend_ledger.jsonl` for the smoke call, passed to every CI job with
    `--spend-from experiments/spend`; `h2h.yaml` model replaced. v1 (`grid.yaml`, PREREG.md
    Amendment 1) is untouched, and its Gemini cells remain NOT RUN.
- **D30 Smoke call (PREREG_v2 Amendment 2, B6) and review fixes.** One request at
  2026-11-15T23:26:29Z UTC, after the Amendment 2 commit 860556b
  (2026-11-15T23:13:54Z, pushed 23:14Z UTC). Synthetic carry_all-shaped prompt (v1
  system prompt plus the invented fixture; 22 messages; no HotpotQA question), sent from the
  local workstation with the key from `.env.local`; labelled `smoke`, never data.
  - **Result.** The response `model` was `gpt-6-luna` (no dated snapshot), on service tier
    `default`; `system_fingerprint` was null. temperature 0, seed 42 and reasoning_effort
    "none" were all accepted (no HTTP 400), so the B3 fallback did not trigger and every run
    sends exactly these parameters. finish_reason was `stop`, and the reply was a valid JSON
    action.
  - **Usage.** prompt 1,693 (cached 0, cache_write 1,690), completion 63 (reasoning 0).
    Cost 0.00024305 USD = (3 x 0.10 + 1,690 x 0.125 + 63 x 0.50) / 1e6, recorded in the
    committed ledger `experiments/spend/spend_ledger.jsonl` (cum_usd 0.000243). The prompt
    bound (6,885) held.
  - **Observation.** On a first request almost the whole prompt is billed as a 30-minute
    cache write (1.25x input). Uncached prefix tokens therefore cost more than the plain input
    rate on this model, and cached reads 0.1x. This is recorded per request (B5) and does not
    change any rule.
  - **Independent review** of the provider and guard (reviewer subagent, no blocking
    defect). Fixed before any data run:
    - The ledger is now committed, and its test fails rather than skips when it is absent.
    - Unpriced and unconfirmed charges persist into later jobs' prior spend. Each OpenAI
      record carries `usd_worst`; an unpriced record counts at 2x it, never 0. Unconfirmed
      attempts count 1x each. The grid's running spend follows the guard's.
    - An unpriced response is charged at 2x its standard worst case (Fast tier = 2x).
    - Billing stops (`billing_hard_limit_reached`, `billing_not_active`,
      `access_terminated`) halt the whole run like `insufficient_quota`.
    - The job's missing-key check is tied by a test to the reason the grid writes. A
      missing key of any provider (Gemini and DashScope included) now fails the job
      (NOT RUN, never green), as instructed for this experiment.
- **D31 Dev round 0 (PREREG_v2 section 5.1) run in CI; no controller change yet.** Two
  `experiment.yml` h2h dispatches on the [REDACTED: infrastructure] runner (Amendment 2, B8), on the dev split only
  (permutation positions 54-77; no test question touched). Run 43776254575 (git e026133,
  first in chain, dev questions 1-12, 23:42-23:50Z UTC). Run 43777353380 (git 9cf2f18,
  identical agent/experiments code, resumed from the first run's artifact, questions 13-24,
  23:55Z-00:03Z UTC).
  - **Runs.** 192/192 ok (24 questions x 8 cells), 0 errors, 0 NOT RUN, 422 requests, about
    2.2 per run against the planned 5 (so 422 against about 890 planned), because the model
    finishes early.
  - **Every request.** Response model `gpt-6-luna`, service tier `default`, finish reason
    `stop`, and one parameter set (temperature 0, seed 42, effort none; nothing rejected).
    Reasoning tokens 0. No prompt exceeded its worst-case bound; the largest was 2,001 tokens,
    and 83 of 422 requests were at least 1,024 tokens (cacheable).
  - **Spend.** 0.041538 USD for the round (0.020224 + 0.021314); the project total is
    0.041781 USD of 5, smoke included.
  - **Dev table.** n = 24 per point; token columns are totals over the 24 questions;
    USD per question; F1 and EM as fractions.

    | arm | k | EM | F1 | input | cached | cache write | output | requests | USD/q |
    |---|---|---|---|---|---|---|---|---|---|
    | react | 2 | 0.375 | 0.591 | 25,268 | 0 | 0 | 3,850 | 57 | 0.000186 |
    | react | 8 | 0.500 | 0.676 | 33,769 | 6,689 | 11,371 | 3,236 | 49 | 0.000195 |
    | notes_reset | 2 | 0.417 | 0.602 | 23,750 | 0 | 0 | 3,873 | 58 | 0.000180 |
    | notes_reset | 8 | 0.542 | 0.686 | 32,916 | 0 | 16,237 | 3,321 | 49 | 0.000223 |
    | window2 | 8 | 0.458 | 0.663 | 35,740 | 5,749 | 14,183 | 3,382 | 50 | 0.000213 |
    | memgpt | 8 | 0.500 | 0.677 | 36,440 | 3,420 | 16,263 | 3,592 | 52 | 0.000231 |
    | agentless | 8 | 0.500 | 0.651 | 47,858 | 0 | 21,964 | 1,496 | 48 | 0.000253 |
    | pbc (k0 3, k_hi 8, e_max 6000) | 3 | 0.458 | 0.642 | 35,902 | 0 | 3,492 | 4,676 | 59 | 0.000251 |

  - **Dev frontier.** notes_reset k=2, notes_reset k=8 and react k=8.
  - **pbc at its start values (descriptive, dev only; H7 is decided on test).** It is
    dominated by memgpt, notes_reset, react and window2 at k=8. Against the highest-F1
    baseline (notes_reset k=8) its cost ratio is 1.12 (paired bootstrap 95% CI 0.96-1.34)
    and its F1 difference is -4.3 points (CI -13.8 to +1.9). Its grounded-answer rate is 1.0.
    It has the most output tokens (the `cites` field) and the most requests. Its low
    cache-write volume shows that its prompts rarely reach the 1,024-token cache minimum.
  - **Next.** Dev rounds 1-2 (one controller change each, PREREG_v2 section 5.2) have not
    started, and none is made here; this table is the basis for that decision. The test
    split was not run.

## 2026-11-16 (later the same day): PREREG_v3, before any v3 request

[NOT INCLUDED: D32 is not referenced by the released pre-registrations]








- **D33 Cross-run cache confound in v2 dev round 0 (methods finding).** `agent/run_grid.py`
  re-created `random.Random(cell_seed)` for every question, so all 24 dev questions ran the cells
  in one fixed order, and arms whose prompts share a prefix (react, notes_reset, window2 and
  memgpt send the same first two requests) could read each other's implicit-cache writes.
  Recomputed with the registered check of `experiments/analyze_v3.py`
  (`cross_run_cache_reads`) on the committed copy of the run 43777353380 artifact
  (`experiments/offline/v2_dev_round0_run43777353380/`): 12,463 of 15,858 cached tokens
  (78.6%) were read by requests whose own run had no earlier request of at least 1,024 tokens,
  so they are provably cross-run (react 5,579, window2 3,464, memgpt 3,420). This is a lower
  bound and agrees with both judges. Arms with unique prompts (pbc, agentless) got no such
  subsidy, so v2's dev costs favour the loop baselines. Fixed for v3 (PREREG_v3 H1, H2): a
  per-(cell, repeat, invocation) salt as the first system message and a per-question reshuffle;
  `agent/tests/test_v3_harness.py` failed on the v2 runner (one order for all questions; no salt)
  and passes on the fix (`experiments/offline/failfirst_v3.txt`).
- **D34 PREREG_v3 registered** (`experiments/PREREG_v3.md`; the commit that adds it is the
  registration; no v3 request before it). Choices made while writing it:
  - **Test split** = positions 4000-4239 (five 48-question blocks). Every range any earlier
    computation touched is listed in `experiments/offline/TOUCHED_POSITIONS.json` (max 1037).
    The design scripts of `recall_sim*.py`, `assemble_*.py` and `tool_variants.py` took their
    ranges as unlogged CLI arguments, so the test starts far above the reported 1037 rather
    than at 1038. Positions 0-47 (v2's designated test) are not used: a single clean rule.
    The 33 design scripts and the v2 dev artifact were copied from the session scratchpad to
    `experiments/offline/` with original and copy SHA-256 (`MANIFEST.json`; absolute local paths
    replaced by `<LOCAL_PATH>` so the tree passes the privacy scan).
  - **Faithful baselines**: notes removed from react, window2 and memgpt (their sources have no
    notes field; the judges measured notes at 29-32% of output tokens and 10-14% of billed USD
    in v2); react capped at 7 steps (ReAct section 3.2); agentless without v2's cites and ids;
    notes_reset keeps its notes as its memory (MEM1-style). Each mapping is in PREREG_v3
    section 4. ReWOO and a few-shot ReAct are registered NOT RUN (not ready by registration).
  - **One answer contract** for all arms (H3), so no arm has a format advantage; identical
    provider settings (no cache option for any arm); `rag1` and `closed_book` reference cells.
  - **Referee points relayed by the coordinator (by 16 Nov), folded in before registration:**
    3 repeats of every (cell, question) on dev and test with their own salts, variance across
    repeats reported; per-arm cache isolation with each arm's cache hit rate; one model
    statement (v1 never ran, v3 is the comparison); a per-arm table with billed USD including
    write premiums, tokens by kind, cache hit rate, turns, latency and EM/F1 with paired CIs,
    losing arms included; the venue scope stated as a controlled test of the cost mechanism on a
    minimal retrieval agent, with coding agents as a limitation and future work; the test split
    clear of every design-touched position.
  - **H8** keeps v2's H7 form and adds the judges' two tightenings: the F1 non-inferiority rule
    also needs a 95% CI lower bound of at least -5 points, and rule (d) (every published cell
    within 2 F1 points must also be beaten by the 0.70 cost ratio). B* is selected on test.
  - **Concurrency 2** by two shard processes, each admitted against half of the remaining
    budget; **v3 spend stop 3.00 USD** (project total 3.04), sized for 3 repeats.
  - **Not done (optional):** the judges' dev-only comparison of AoB's PRF hop with the
    AnchorQuote anchor/link hop was not run; AoB keeps the PRF hop as specified.
  - **Fail-first**: 6 harness tests against the v2 runner, the split test with a block moved onto
    a touched range, and 23 single mutations of the AoB, harness and analysis code, each killed
    (`experiments/offline/failfirst_v3.txt`). The suite was 195 tests before v3.
- **D35 PREREG_v3 Amendment 1 (before any v3 request).** An independent read-only review
  (reviewer subagent) of the registration commit dc9634d confirmed the salt, AoB, split and H8
  logic and found operational issues, all fixed before the first request: the block pre-check
  would have refused test dispatches (estimate now 1,500/150 tokens per run from v2 dev round 0,
  counted over remaining runs only); a billing stop in one shard did not stop the other (now a
  shared halt file that also blocks later dispatches until the author clears it); `questions`
  was applied per shard (now a total, before sharding); the inclusive -2/-5-point thresholds
  were not float-safe (tolerance 1e-9); a retried run reused its failed attempt's salt (attempt
  now in the salt); a partly run cell could become B* (now needs runs on at least 95% of the
  questions). Section 9's "13 cells" is corrected to 15. The new tests failed on the dc9634d
  code (`experiments/offline/failfirst_v3.txt`, Amendment 1 section) and pass now.
- **D36 First v3 smoke attempt: the API key is invalidated (BLOCKED).** Dispatched after
  Amendment 1 (`experiment.yml` job `h2h3`, split `smoke`, run 43805900879, commit 8d8bc79,
  04:44:36Z UTC; resumed from the v2 artifact of run 43777353380, so the job's prior spend was
  the correct 0.041781 USD). The single request returned HTTP 401 `token_invalidated` ("Your
  API key has been invalidated"). It was not billed: spend stays 0.041781 USD, and no reply or
  data exists. The `OPENAI_API_KEY` secret was last set 2026-11-15T23:05Z UTC and worked for
  v2 dev round 0 (D31, until 00:03Z on 16 Nov), so the key was revoked or rotated after that. A
  different secret, [REDACTED: infrastructure], was created at 04:36Z on 16 Nov, [REDACTED: unrelated]
  [REDACTED: unrelated]. It is not used: the registered key is `OPENAI_API_KEY`, and changing
  credentials needs the author. PREREG_v3 Amendment 2 makes an HTTP 401 halt the whole run (a
  new `ProviderAuthInvalid`; its test failed on the old provider). Dev round 0 cannot start
  until the author sets a valid `OPENAI_API_KEY`.
[NOT INCLUDED: D37 is not referenced by the released pre-registrations]



- **D38 New key in the Actions secret; branch rebased onto the rewritten main (PREREG_v3
  Amendment 3; no design change).** The author approved resuming v3 (via the coordinator).
  - **Key.** The Actions secret `OPENAI_API_KEY` was reset at 2026-11-16T05:13Z UTC. It now holds
    a different key on the same account type; the earlier key was invalidated (D36). Provider,
    model, parameters and prices are unchanged. Spend carries over (0.041781 USD), and the v3
    stop stays at 3.00 USD. The 401 halt (Amendment 2) stays.
  - **History.** `main` was rewritten on the author's instruction to remove AI co-author
    trailers (trees identical; new head 5baf009), and the v3 work was rebased onto it and published as the new branch exp/agent-rag-v3b. The
    old branch exp/agent-rag-v3 was left as it was, because a force-push was not permitted.
    The registration commit dc9634d is now bfcde41, with its author date (the registration
    time, 2026-11-16T04:31:15Z) preserved. The full SHA map is in Amendment 3.
  - No further request was made between D36 and this entry.
[NOT INCLUDED: D39 is not referenced by the released pre-registrations]









[NOT INCLUDED: D40 is not referenced by the released pre-registrations]




- **D41 v3 dev round 0 (PREREG_v3 section 6.2) done in CI; every cell, 3 repeats.** Two
  `h2h3` dispatches, split dev, run_label r0, two shards each.
  - Run 43810725916 (commit 0f9cd45; questions 1-48; 05:34-07:04Z UTC; 0.362152 USD).
  - Run 43821376770 (commit 93b875c, identical agent and experiment code; resumed; questions
    49-96; 07:06-08:50Z UTC; 0.351395 USD).
  - **Runs and requests.** 4,320 runs (15 cells x 96 questions x 3 repeats), all ok: 0 errors,
    0 NOT RUN, 0 parse errors. 7,919 requests. Every request returned model `gpt-6-luna`, tier
    `default` and finish reason `stop`, with 0 reasoning tokens and no parameter rejected.
  - **Cache isolation check:** 0 provable cross-run cached tokens. The cached reads that remain
    (react, window2, memgpt) are each run's own earlier prefix.
  - **Spend.** Dev round 0 cost 0.713547 USD. The project total is 0.755364 USD, against a v3
    stop of 3.04.
  - **Dev table.** Token columns are totals over the 288 runs; USD is per question and run;
    EM/F1 are means of the repeat-averaged per-question values. "neutral" means every input
    token at 0.10; "cache hit" means cached / input.

    | arm | k | n | EM | F1 | requests | input | cached | cache write | output | USD/q | neutral USD/q | cache hit | latency s |
    |---|---|---|---|---|---|---|---|---|---|---|---|---|---|
    | agentless | 8 | 288 | 0.622 | 0.728 | 576 | 500,256 | 0 | 138,761 | 11,524 | 0.000206 | 0.000194 | 0.000 | 3.81 |
    | aob | 6 | 288 | 0.587 | 0.717 | 304 | 239,525 | 0 | 0 | 5,520 | 0.000093 | 0.000093 | 0.000 | 5.48 |
    | aob_llmq | 6 | 288 | 0.573 | 0.706 | 588 | 263,646 | 0 | 0 | 12,343 | 0.000113 | 0.000113 | 0.000 | 3.89 |
    | aob_nogate | 6 | 288 | 0.566 | 0.698 | 288 | 227,193 | 0 | 0 | 5,187 | 0.000088 | 0.000088 | 0.000 | 1.85 |
    | aob_nohop | 6 | 288 | 0.566 | 0.713 | 302 | 238,084 | 0 | 0 | 5,500 | 0.000092 | 0.000092 | 0.000 | 1.67 |
    | aob_nopack | 6 | 288 | 0.580 | 0.728 | 294 | 295,565 | 0 | 139,409 | 5,251 | 0.000124 | 0.000112 | 0.000 | 1.86 |
    | closed_book | 0 | 288 | 0.382 | 0.501 | 288 | 32,412 | 0 | 0 | 4,121 | 0.000018 | 0.000018 | 0.000 | 4.02 |
    | memgpt | 8 | 288 | 0.597 | 0.721 | 686 | 489,864 | 3,494 | 211,307 | 32,248 | 0.000243 | 0.000226 | 0.007 | 5.23 |
    | notes_reset | 2 | 288 | 0.580 | 0.701 | 828 | 378,403 | 0 | 0 | 56,210 | 0.000229 | 0.000229 | 0.000 | 10.31 |
    | notes_reset | 8 | 288 | 0.601 | 0.731 | 653 | 474,730 | 0 | 247,100 | 43,093 | 0.000261 | 0.000240 | 0.000 | 5.34 |
    | pbc | 3 | 288 | 0.618 | 0.749 | 760 | 496,453 | 0 | 41,590 | 58,912 | 0.000278 | 0.000275 | 0.000 | 6.16 |
    | rag1 | 8 | 288 | 0.559 | 0.668 | 288 | 222,900 | 0 | 3,588 | 4,204 | 0.000085 | 0.000085 | 0.000 | 1.76 |
    | react | 2 | 288 | 0.573 | 0.715 | 779 | 368,256 | 10,942 | 18,687 | 34,924 | 0.000187 | 0.000188 | 0.030 | 5.98 |
    | react | 8 | 288 | 0.608 | 0.734 | 647 | 498,484 | 51,349 | 212,627 | 29,026 | 0.000226 | 0.000223 | 0.103 | 4.93 |
    | window2 | 8 | 288 | 0.608 | 0.730 | 638 | 473,044 | 21,029 | 220,565 | 28,735 | 0.000227 | 0.000214 | 0.044 | 4.85 |

  - **Per-cell 95% CIs and repeat variance** (F1 CI; SD of the three per-repeat F1 means;
    answer agreement across repeats):
    - aob 0.637-0.792; SD 0.006; agreement 0.83.
    - react k=8 0.658-0.806; SD 0.010; agreement 0.77.
    - notes_reset k=8 0.654-0.803; SD 0.002; agreement 0.78.
    - window2 0.655-0.802; SD 0.028; agreement 0.82.
    - The other cells are in the evidence JSON of the run artifact.
  - **Descriptive dev check against B\*_dev = react k=8 (F1 0.734); decisions are made on test.**
    - aob/B\* cost ratio 0.411 (CI 0.379-0.445).
    - F1 difference -1.67 points (CI -6.28 to +2.50).
    - Rules (a), (c) and (d) hold. Rule (b) fails only on the CI lower bound (n = 96).
    - The billed frontier includes aob, its ablations, rag1, closed_book, agentless, react k=8
      and pbc.
  - **Ablations (aob vs X):**
    - nogate: ratio 1.06, F1 +1.90.
    - nohop: ratio 1.01, F1 +0.41.
    - nopack: ratio 0.75, F1 -1.09.
    - llmq: ratio 0.82, F1 +1.13.
    - rag1: ratio 1.09, F1 +4.95.
    - closed_book: F1 +21.7.
    - pbc: ratio 0.33, F1 -3.19.
    - Every F1 CI includes 0 except closed_book.
  - **Gate.** 1.056 answer requests per run. Commits: 95.1% at request 1, 4.2% at request 2,
    0.7% at request 3. Supported share 0.889. F1 0.777 when supported, 0.237 when not. The
    largest aob prompt was 963 tokens; the median was 790.
  - **Registered triggers (section 6.3), evaluated on all 96 questions:**
    - T1 did not fire: the F1 gap is -1.67 points, not below -2.0.
    - T2 did not fire: 0% of aob requests were at or above 1,024 tokens, and the median of 790
      is not under 750.
    - Defect check: 0 parse errors. But 31 of 304 aob requests (10.2%) failed the gate and
      returned no `next` query (30 empty lists, 1 missing key). The registered format asks for
      "one or two short search queries". Those runs could not escalate: their mean F1 is 0.244,
      77% are below 0.5, and only 16 escalations happened in 288 runs. This is a reply-format
      failure above the registered 5%, so the defect option applies (D42).
- **D42 Dev round 1 (PREREG_v3 section 6.3, defect option): one change to aob, recorded before
  the round runs.**
  - **The change, in one place:** the word "always" is added to AoB's `next` instruction. The
    full line now reads: `- "next": always one or two short search queries for the fact your
    answer depends on most; they are used only if the passages do not state your answer.`
  - The same word is added to the registered T1 variant (`AOB_THOUGHT_PROMPT`), so that T1, if
    it fires later, still differs from aob by the thought field only.
  - Nothing else changes. The ablations inherit it, as they share `AOB_PROMPT`.
  - **Reason:** D41's defect check. The reply format requires one or two queries, and 10.2% of
    aob requests gave none after a failed gate, which disabled the batch escalation.
  - **Round 1 setup:** aob only, 3 repeats, all 96 dev questions, run_label r1. Baselines are
    not re-run; they are compared from round 0. T1 and T2 are then re-evaluated on the round-1
    aob data. "No change twice" holds.
- **D43 Dev round 1 result; no round 2; FREEZE (PREREG_v3 section 6.3-6.4).** Run 43835072503
  (commit 7869033; aob only; 3 repeats; 96 dev questions; run_label r1; 08:59-09:04Z UTC).
  - **Runs.** 288 runs and 309 requests, all ok, with 0 parse errors. Every request returned
    `gpt-6-luna` and `stop`. Cost 0.027337 USD; the project total is now 0.782702 USD.
  - **aob in round 1:** EM 0.615, F1 0.742 (CI 0.669-0.812). Repeat F1 means 0.743, 0.744 and
    0.738 (SD 0.003). Over 288 runs: input 243,697 tokens, 0 cached, 0 cache write, output
    5,936. Cost 0.000095 USD per question. Largest prompt 964 tokens, median 790.
  - **Against round-0 B\*_dev = react k=8 (descriptive; the decision is on test):** cost ratio
    0.420 (CI 0.388-0.455), F1 +0.82 points (CI -3.31 to +4.90). Rules (a)-(d) all hold on dev.
    Rule (d) cells:
    - agentless: 0.461 (CI upper 0.485);
    - notes_reset k=8: 0.364 (0.396);
    - react k=8: 0.420 (0.455);
    - window2: 0.419 (0.460).
  - **Gate.** 1.073 answer requests per run. Commits: 93.8% at request 1, 5.2% at request 2,
    1.0% at request 3. Supported share 0.910. Escalations rose from 16 to 21. A failed gate with
    no `next` query fell from 31 of 304 requests to 25 of 309 (8.1%).
  - **Triggers re-evaluated on the round-1 aob data:**
    - T1 did not fire (the F1 gap is +0.82 points).
    - T2 did not fire (0% at or above 1,024 tokens; median 790).
    - The defect option was used in round 1 and is not applied twice. The remaining 8.1% is
      reported as a property of the frozen agent, not tuned further.
    - So there is no dev round 2.
  - **Frozen design.** The aob parameters are as registered (k1 6, m2 4, budget 3,500, r_max 3,
    n_next 2, m3 4, hop on, LLM query off, thought off), with D42's one-word prompt change. The
    four ablations inherit it. The baselines, references, pbc, harness, splits, H8 and the
    analysis are unchanged from PREREG_v3 with Amendments 1-3. The freeze commit is the one
    that adds this entry, tagged `prereg-v3-freeze`.
  - **Test plan (not run).**
    - Every one of the 15 cells, 3 repeats, test positions 4000-4239 (240 questions).
    - Five `h2h3` dispatches with `--ref prereg-v3-freeze`, split test, `questions` = 48, 96,
      144, 192 and 240. The first resumes from run 43835072503; each later one resumes from the
      previous dispatch.
    - About 19,800 requests (27.5 per question and repeat in dev round 0).
    - About 1.78 USD, at dev round 0's 0.00248 USD per question and repeat. The project total
      would then be about 2.57 USD, under the 3.04 stop and the 5.00 cap.
    - About 95 minutes per dispatch at concurrency 2 (dev round 0 took 91 and 104 minutes per
      48 questions).
    - Cut order (PREREG_v3 section 6.5) if late. The block pre-check admits each dispatch:
      about 0.28 USD per shard against at least 0.41 USD of headroom at the last dispatch.
- **D44 PREREG_v3 Amendment 4: operational fixes after an independent audit of the freeze
  e274b76.** The auditor's verdict was GO-WITH-CONDITIONS, with no method blocker; it
  reproduced D41 and D43 exactly. The orchestrator chose a minimal orchestration fix and a
  re-freeze, before any test request.
  - **Fixes (E1-E4 of the amendment):**
    - `runs.jsonl` is appended and flushed per finished run.
    - The container is stopped and its outputs copied out on any exit (a trap, plus an
      `always()` step before the upload; run-step timeout 165 below the job's 180 minutes).
    - A job refuses to resume from a hollow artifact (`experiments/ci/check_prev_v3.py`).
    - Spend-guard charges without a request record are written as `unrecorded_charge` records
      and counted in prior spend; `experiments/attempts_v3.py` reports failed and unconfirmed
      attempts per arm.
  - **Fail-first:** each of the 5 new tests (`agent/tests/test_v3_operational.py`) failed on
    the e274b76 versions of `run_grid.py`, `job.sh`, `run_in_container.sh` and `experiment.yml`
    (`experiments/offline/failfirst_v3.txt`). 272 tests pass.
  - **No design change: git blob hashes at e274b76 and at the re-freeze are identical.**
    - `agent/arms.py` 0423b20b0c273ce478c5d78beb2dd7822566c55a
    - `agent/agent.py` 4e60990332528282432dc2e641101da29b6da0df
    - `agent/prompts.py` 449bd6ea182900443a462497bf92be7e85b92c8c
    - `agent/data.py` 4631f1382926655f16147c6b70d9e6354537bba7
    - `experiments/configs/h2h_v3.yaml` 2d8caee6f76b38d7969a1926725f31912465af4b
    - `experiments/analyze_v3.py` 1143f99f6c93d685274c9309b19fee28ce320b23
    - The run configuration hash is unchanged (sha256 of `h2h_v3.yaml` a6384cc63fe6d917...).
- **D45 Test procedure rules (orchestrator, before any test request).**
  - **C1 Recovery rule.** Never resume from a failed dispatch. Re-run its block once from the
    last good artifact. Add the failed dispatch's worst-case spend through `prior_spend_usd`
    (its records may be lost), and report it.
  - **C2 One dispatch at a time, verified after each.**
    - Use 24-question increments: 10 dispatches with `questions` = 24, 48, ..., 240.
    - After each, check that the run's head SHA and `ci_context.txt` show the freeze SHA, and
      that the inputs are exactly as planned.
    - Check that the ok-run count is 1,080 x k after step k (24 questions x 15 cells x 3
      repeats), that there is no HALT file, and that prior spend has not decreased.
  - **C3 No interim decisions.** Every dispatch runs whatever the interim H8 printouts say.
    "Late" means 21 Nov 2026 01:00 UTC, after which the registered cut order applies. If the
    block pre-check refuses the last block, the result is reported at the n reached.
  - **C4** `OPENAI_API_KEY` is not changed during the test.
- **D46 Errata.**
  - (1) The first smoke attempt (D36, run 43805900879) made **2** unbilled requests, both
    HTTP 401, not 1: its run errored and was retried once at the end of the grid.
  - (2) Unrecorded so far: dispatch 43807166607 (job `tests`, commit 15db707, no API request;
    265 tests passed in the container).
  - (3) PREREG_v3 Appendix A shows the pre-D42 AoB prompts. The frozen strings add "always"
    (D42), and Amendment 4 points readers there.
- **D47 Test-run failure rules (orchestrator, after an independent re-audit of e274b76 to
  f6bb418 gave GO-WITH-CONDITIONS; the author authorised the test).** Committed before test
  dispatch 1. Docs only; the tag `prereg-v3-freeze2` stays on f6bb418, which every test
  dispatch runs (`--ref prereg-v3-freeze2`).
  - (1) **Failed dispatch.** A dispatch has failed if its job is not green, or if the next job's
    resume check would refuse it. That includes a block with any errored or NOT RUN run. Its
    artifact is kept and listed, its results are excluded, and its block is re-run once from the
    last good artifact.
  - (2) **Spend carry-forward.** Every later dispatch carries a cumulative manual
    `prior_spend_usd` for each failed dispatch: its charged spend from its own records, plus 2x
    the largest `usd_worst` among requests in flight. If its records are lost, the amount is
    0.2835 USD (the pre-check estimate for one 24-question block).
  - (3) **Second failure.** If the same block fails twice: HALT and report BLOCKED.
  - (4) **[REDACTED: infrastructure] or runner crash.** First `docker cp` the outputs from the stopped container, then
    choose the worst-case amount.
  - (5) **Post-test checks:**
    - the auditor's first-report checklist: dataset hash and split positions; one ok run per
      (cell, question, repeat); provenance SHAs and salts; model, tier and finish reason per
      request; cache isolation and write-field presence; USD recomputed from usage; defect and
      gate statistics; reproduction of the CI analysis;
    - a list of every `unrecorded_charge` record;
    - confirmation that there are no orphan requests;
    - AoB gate statistics recomputed without `unrecorded_charge` rows.
[NOT INCLUDED: D48 is not referenced by the released pre-registrations]
















- **D49 PREREG_v3 Amendment 5: the v3 spend stop goes from 3.04 to 3.50 USD (project total).**
  Corrected, docs only, before any request under it (auditor re-check of c666e81).
  - **Scope.** Budget only. Author decision (a), relayed by the orchestrator during the test,
    before dispatch 9's first request.
  - **Options given to the author:** (a) raise the stop to 3.50 USD; (b) keep 3.04 USD and stop at
    the n reached.
  - **Basis: cost trend only.**
    - Dispatches 1-5 cost 0.187, 0.197, 0.235, 0.237 and 0.221 USD, inflated by worst-case
      charges for 46 unconfirmed (timed-out) attempts.
    - The pre-check estimates 0.14175 USD per shard (540 runs x 0.0002625; 0.2835 for both
      shards). Under 3.04 it refuses once prior spend exceeds 2.7565 USD.
    - So dispatch 10 is refused if dispatches 6-9 average about 0.225 USD or more, and dispatch
      9 if 6-8 average about 0.30 USD or more.
    - My earlier note (that dispatch 9 itself would be refused at 0.2835 per shard) used the
      two-shard figure per shard and was wrong. The purpose, completing n = 240, is unchanged.
  - **Not consulted.** Interim H8 printouts existed in the job logs (the job's analysis step) and
    were not consulted by anyone in this decision.
  - **Supersedes D45 C3's fallback** ("report at the n reached") for this test. That fallback was
    for an unforeseen budget refusal; this one was foreseen from spend alone and resolved by the
    author beforehand.
  - **Why a file changes.** The stop is the config constant `budget.max_usd`. `job.sh` and
    `run_grid.py` take the minimum of it and any dispatch input, by design, so no input can
    raise it.
  - **Files changed against f6bb418 (prereg-v3-freeze2)**, with git blob hashes f6bb418 ->
    freeze3:
    - `experiments/configs/h2h_v3.yaml`: 2d8caee6f76b -> 7c84a86a4656. The only runnable change
      is one line, `max_usd: 3.04` -> `max_usd: 3.50`. The file's sha256 goes from a6384cc6...
      to 370eaee3274e07d1...
    - `experiments/PREREG_v3.md` (Amendment 5) and `experiments/DECISIONS.md` (D47-D50): docs.
  - **Unchanged** (same blob at f6bb418 and freeze3):
    - `agent/arms.py` 0423b20b0c27
    - `agent/agent.py` 4e6099033252
    - `agent/prompts.py` 449bd6ea1829
    - `agent/data.py` 4631f1382926
    - `agent/run_grid.py` 7b6197d1b43f
    - `agent/providers.py` f8b0ced017bf
    - `agent/budget.py` 5882634ae701
    - `agent/retrieval.py` 53991ad855fd
    - `agent/metrics.py` e31333b2774d
    - `experiments/analyze_v3.py` 1143f99f6c93
    - `experiments/ci/job.sh` af8564ef495e
    - `experiments/ci/run_in_container.sh` a1015ea82410
    - `experiments/ci/check_prev_v3.py` 40f088b6f6c6
    - `experiments/attempts_v3.py` c896b50fc6c0
    - `.github/workflows/experiment.yml` 1fccdeedc010
  - **Mechanism and audit.** The auditor checked c666e81: GO-WITH-CONDITIONS, the one-line
    change clean, mixed-config runs pool correctly. The tag `prereg-v3-freeze3` is on the
    corrected docs-only commit that follows c666e81; `git diff --name-only` between the two
    lists only `.md` files. Dispatches 9-10 run from it. Their C2 checks also confirm that head
    SHA and that `h2h3_budget.txt` reads "v3 spend stop 3.5".
[NOT INCLUDED: D50 is not referenced by the released pre-registrations]





[NOT INCLUDED: D51 is not referenced by the released pre-registrations]



















[NOT INCLUDED: D52 is not referenced by the released pre-registrations]

































































































[NOT INCLUDED: D53 is not referenced by the released pre-registrations]













