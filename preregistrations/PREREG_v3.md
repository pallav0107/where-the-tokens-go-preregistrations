# Pre-registration v3: Answer-or-Batch (AoB) against faithful published agents

Status: **PRE-REGISTERED**. The commit that first adds this file is the registration
timestamp. When it was written no v3 request of any kind (smoke included) had been made; the
last API request of this project was v2 dev round 0 (D31, 2026-11-16 00:03Z UTC). It
supersedes `experiments/PREREG_v2.md` (with Amendment 2) for every purpose except the record
of what v2 did (section 10). Deviations after this commit go to `experiments/DECISIONS.md`.

## 0. What has and has not run (one model statement)

- **v1** (`experiments/PREREG.md`, Gemini models, Amendment 1) **never ran**: no Gemini key ever
  existed (D13, D19, D25). Its hypotheses H1-H6 are NOT RUN and are reported as such.
- **v2** (PREREG_v2, moved to OpenAI `gpt-6-luna` by Amendment 2 before any call) ran its
  smoke call (D30) and dev round 0 only (D31). Its controller `pbc` was dominated on dev; H7 was
  never evaluated and the v2 test split never ran. v2 is superseded here (section 10).
- **v3** (this file), on `gpt-6-luna`, is the comparison the paper reports.

## 1. Goal and scope

Compare, on one model, one task and identical provider settings, the billed cost and accuracy
of **AoB**, a turn-priced controller derived from this paper's cost model (cost = context x
turns, priced per request with the provider's cache-write and cache-read rates), against
faithful re-implementations of published agents and two reference points. Report the
cost-accuracy frontier for every arm, including arms where AoB loses.

Scope, stated plainly: this is a **controlled test of the cost mechanism on a minimal retrieval
agent** (BM25 search, read, finish) on HotpotQA multi-hop questions. It is not a coding-agent
benchmark. Runs here last about two requests, so the superlinear re-carry term of the cost model
is not exercised; generalisation to software-engineering agents (long trajectories whose tool
outputs are builds, tests and diffs) is a limitation and future work, and the paper says so. No
claim beyond this benchmark under these conditions; no "best" or "state-of-the-art" wording.

## 2. Data and splits (frozen now)

- Dataset: HotpotQA distractor validation (Hugging Face `hotpotqa/hotpot_qa`, SHA-256
  `c20b638ca82b21d04fe12e14ff417ad05153d4d215a65de54497fca4e972f7c6`, CC BY-SA 4.0), rows sorted
  by id; positions index `numpy.random.default_rng(20261115).permutation(7405)` (as v1/v2).
- Questions are grouped in **48-question blocks**; each block's knowledge base pools the
  context paragraphs of its own 48 questions (v1 section 4 rule), so retrieval difficulty is the
  same in every block.
- **Dev** = positions 54-149: blocks 54-101 and 102-149 (96 questions). Dev positions were
  already used by design-stage statistics; dev is for tuning only.
- **Test** = positions **4000-4239**: blocks 4000-4047, 4048-4095, 4096-4143, 4144-4191,
  4192-4239 (240 questions). Touched exactly once, from the freeze SHA (section 6).
- **Excluded from test (named):** every range in `experiments/offline/TOUCHED_POSITIONS.json`:
  0-47 (v1 sample and v2's designated test; never run in v2, excluded by this clean rule),
  48-53 (v1 smoke), 54-77 (v2 dev, LLM runs, D31), 54-461 (design scripts `hop2.py`,
  `pack.py`, `retr1.py`, `calib_v3.py`, `support1/2.py`, `reconstruct.py`,
  `dev_questions.py`, with gold labels), 78-1037 (design scripts `recall_sim.py`,
  `recall_sim2.py`, `assemble_sim.py`, `assemble_tool_only.py`, `tool_variants.py`; their
  start/end were CLI arguments that were not logged, and both design judges report 78-1037),
  102-197 (designer A's simulated blocks), and 54-149 (v3 dev). The largest touched position
  is 1037; the test starts 2,962 positions above it as a margin against an unlogged larger CLI
  range. The scripts and the v2 dev artifact are copied, with SHA-256 hashes, to
  `experiments/offline/` (`MANIFEST.json`). `agent/tests/test_v3_splits.py` asserts the
  disjointness; it was shown to fail when a test block is moved onto a touched range.
- The design-stage MuSiQue statistics (`musique_stats.py`) concern another dataset that v3 does
  not use.

## 3. Shared harness (every arm, identically)

- **H1 Cache isolation.** Every request begins with the system message `session <salt>`,
  16 hex characters = SHA-256(`nonce|arm|k|repeat`), where `nonce` is a fresh random 128-bit
  value per grid invocation (recorded in `provenance.json`). The salt is constant within one
  (cell, repeat) of one invocation and differs between cells, repeats and invocations, so no
  request can share a cacheable prefix with another arm's, another repeat's or an earlier
  round's requests. (The motivation, D33: in v2 dev round 0, 12,463 of 15,858 cached tokens,
  78.6%, were provably read across runs between arms with identical prefixes.)
- **H2 Order.** Question-major; for each question every (cell, repeat) pair runs in an order
  re-shuffled per question by `random.Random("20261117:<question id>")`.
- **H3 Answer contract.** The sentence in Appendix A (`ANSWER_CONTRACT`) appears verbatim in
  every arm's prompt. Answers are scored identically (official HotpotQA normalisation, EM and
  F1). This is a declared harness deviation from the v2 and source prompts, applied to all arms
  so that no arm has a format advantage.
- **H4 Provider configuration (PREREG_v2 Amendment 2, unchanged).** OpenAI Chat Completions,
  `gpt-6-luna`, temperature 0, seed 42, `reasoning_effort` "none", JSON mode,
  `max_completion_tokens` 4096, `store` false, `service_tier` "default"; implicit caching only:
  no `prompt_cache_key`, `prompt_cache_options`, retention or breakpoint for any arm (B4). The
  B2 billed-USD formula, B3 fallback rule, B5 metric mapping and B7 guards apply unchanged.
- **H5 Repeats.** Temperature 0 is not deterministic on this model (v2 dev: identical prompts
  gave different replies), so every (cell, question) runs **3 independent repeats** on dev and
  on test, each with its own salt. The question is the statistical unit; repeats are averaged
  within a question (section 8), and variance across repeats is reported.
- **H6 Concurrency 2.** A job runs two processes on the two halves of the question list
  (`--shard 0/2`, `1/2`, by index); each is admitted against half of the budget remaining
  under the v3 spend stop (section 9), so together they cannot exceed it.
- **H7 Turn caps.** Harness cap 10 agent steps; an arm's source setting overrides it (react: 7).
  Helper requests (memgpt summariser, aob_llmq query) count as requests and in cost.

## 4. Arms (settings frozen now; `experiments/configs/h2h_v3.yaml`)

Published baselines, **the only B\*-eligible cells** (each mapping cited in the paper):

| cell | source and mapping | settings |
|---|---|---|
| `react` k=2, k=8 | ReAct (Yao et al. 2023, `yao2023react`): thought, action, observation; full trajectory re-sent; no memory or notes field. The source uses 7 steps for HotpotQA (ReAct section 3.2: "We set 7 and 5 steps for HotpotQA and FEVER"). Zero-shot instruction instead of the 6 few-shot exemplars, whose Wikipedia Search/Lookup tools do not exist here (declared). | ReAct prompt (App. A), max 7 steps |
| `notes_reset` k=2, k=8 | Constant-memory agent in the style of MEM1 (Zhou et al. 2025, `zhou2025mem1`): the agent rewrites an internal state each step and only that state plus the latest action/observation are kept. Prompted, not RL-trained (declared). The notes field is this arm's memory, so it stays (v1/v2 wording, at most 60 words). | notes prompt, max 10 |
| `window2` k=8 | Fixed sliding window: the last 2 action/observation pairs, no memory field. Closest published policy: observation masking (Lindenbauer et al. 2025, `lindenbauer2025complexity`), which keeps the reasoning/action history and masks old observations; window2 drops whole pairs (declared). | ReAct prompt, max 10 |
| `memgpt` k=8 | MemGPT queue manager (Packer et al. 2023, `packer2023memgpt`, section 2.2): on overflow about half of the messages are evicted into a recursive summary written by an LLM request; the main agent has no notes field. Trigger 6,000 characters of summary plus pairs (v2); the source triggers at a fraction of the context window, which never fires here (declared). | ReAct prompt, max 10, summariser as v2 |
| `agentless` k=8 | Agentless (Xia et al. 2024, `xia2024agentless`): fixed phases, the LLM never chooses actions. Retrieve for the question; request 1 writes one follow-up query; retrieve (new chunks only); request 2 answers from both sets. v2's `cites` field and chunk ids are removed (not in the source). | exactly 2 requests |

Reference points (never B\*): `rag1` k=8, one retrieve-then-read request (agentless's answer
request over the question's top-8 chunks); `closed_book`, one request with no passages.

The agent: `aob` (section 5). One-variable ablations (descriptive, never B\*), each the frozen
aob with exactly one parameter changed: `aob_nogate` (r_max = 1: always commit after request 1;
isolates the gate and batch), `aob_nohop` (hop off: question-only top-10 at the same budget;
isolates the PRF hop), `aob_nopack` (no character budget: all prefetch and follow-up chunks are
shown; isolates the sub-1,024 packing rule; its billing share is also isolated without extra
runs by the repricings of section 8), `aob_llmq` (one extra LLM request writes the first query,
which replaces the question in prefetch; isolates the step AoB removes).

Superseded: `pbc` (v2 controller) at its v2 start values (k0 3, k_hi 8, e_max 6,000) with the
shared answer contract; reported as superseded, never B\*.

NOT RUN (registered now, reported as such): ReWOO (not implemented by the registration
cut-off), a few-shot ReAct with converted exemplars (conversion not verified in time),
Reflexion (as v2).

## 5. AoB (Answer-or-Batch)

Parameters (chosen from retrieval statistics alone, no LLM call): k1 = 6, m2 = 4, character
budget B = 3,500, r_max = 3, at most n_next = 2 follow-up queries with m3 = 4 new chunks each.
No notes, cites or thought field.

1. **Prefetch, no LLM call.** E1 = top-k1 chunks for the question. PRF hop: E2 = the first m2
   chunks of the search for `question + " " + E1[0].text` at depth k1+m2+6 whose document is
   not in E1.
2. **Pack under the cache minimum.** Priority E1[0], E1[1], E2[0], then E2[i] and E1[j]
   alternating. A chunk (its `Title: text` line, no id) is appended only if the request's
   characters (system prompt, question, passages, retry line; the 24-character salt message is
   outside B) stay within B; otherwise it is skipped and the next is tried. On dev (offline, no
   LLM) request 1 has a median of 3,392 characters and 8 passages, about 820 tokens at the
   v2-calibrated 4.14 characters per token and at most 979 even at 3.6, below the 1,024-token
   cache minimum, so no request pays the 1.25x cache-write rate. This is cache admission done
   by prompt size, with no API option (H4 holds for every arm).
3. **Request r.** System `AOB_PROMPT`; user `Question: {q}` + blank line + `Passages:` + the
   packed lines; for r > 1 also a blank line and `AOB_RETRY_LINE` with the previous answer.
   Reply `{"answer": ..., "next": [...]}`.
4. **Gate, no LLM call.** Commit if the answer's official normalisation is yes/no, or its token
   sequence occurs contiguously in the normalised text of the passages shown in request r.
   Also commit (supported = false) when r = r_max or when the follow-up queries return no chunk
   not shown before. An unparseable reply counts as an empty answer.
5. **Batch.** All `next` queries (at most 2) run in one controller step: for each, the first m3
   chunks of its search at depth k1+m3+6 not shown before (nor taken by the earlier query).
   The next pack's priority: the first new chunk of each query, the previous pack's first three,
   the remaining new chunks round-robin, then the rest of the previous pack, under B. Go to 3.

Each request logs its gate outcome, commit flag, passages, characters and queries.

What is claimed as new: the answer and the next queries come back in one reply and a free span
check picks the branch (a stop/continue rule with no extra call, no token probabilities, no
verifier model, no training), and the evidence window is sized from the price schedule (cache
minimum and write premium). Not new and not claimed: retrieval before the first request (as in
IRCoT and Agentless), the PRF hop, batched queries, adaptive stopping as a general idea.

## 6. Procedure

1. **Smoke** (after this commit, before any data): one aob request on the invented fixture
   `agent/tests/fixtures/hotpot_tiny.jsonl` (no HotpotQA question), `h2h_v3_smoke.yaml`,
   r_max 1, through CI. Labelled `smoke`, never data; it checks the salt message, the reply
   format and the prompt size. Its cost enters the spend chain.
2. **Dev round 0**: every cell of section 4, 3 repeats, on dev, in CI (`experiment.yml`, job
   `h2h3`, split `dev`, [REDACTED: infrastructure] runner), resuming from the smoke job's artifact. The v3 chain's first
   job (the smoke) resumes from the v2 artifact of run 43777353380, so every earlier USD is
   counted (the committed ledger holds only the v2 smoke call).
3. **Dev rounds 1-2** (aob only, 3 repeats; baselines are never tuned or re-run). After each
   round the triggers are evaluated on the latest aob dev data against B\*_dev of round 0, and
   at most one change is made per round, in this order:
   - T1 (accuracy): if F1(aob) - F1(B\*_dev) < -2.0 points, set `thought: true`
     (`AOB_THOUGHT_PROMPT`, Appendix A).
   - T2 (size): if more than 10% of aob requests have at least 1,024 prompt tokens, B -= 300; or
     if the median aob prompt is under 750 tokens, B += 200.
   - Otherwise one change that fixes a defect seen in dev round 0 (a bug, or a reply-format
     failure in more than 5% of aob requests), with its evidence recorded in DECISIONS.md before
     the round runs.
   No change is applied twice; no trigger -> no round. The ablations inherit the frozen aob
   parameters (they are defined as one-parameter changes of it).
4. **Freeze**: record the commit SHA, the test plan and its estimated cost; then the
   orchestrator reviews the frozen design. The test runs once, from the freeze SHA, for every
   cell with 3 repeats, in dispatches of at most about 2.5 hours that resume one another.
5. **Cut order if late** (fixed now): (i) repeats 2-3 of the four ablations and pbc on test;
   (ii) pbc on test; (iii) the fifth test block (n = 192). H8's cells (published baselines,
   references, aob) always keep 3 repeats.
6. Errors: as v2 (retry once at the end; 3 consecutive errors stop a cell; NOT RUN records).

## 7. Outputs

Raw JSONL in Actions artifacts (90-day retention); `experiments/analyze_v3.py` writes the
evidence envelope (`kind: agent_h2h_v3`, figures prefixed `Hthree`) and a per-arm table;
paper macros are generated from the evidence file.

## 8. Hypothesis H8 and statistics

Cost = mean billed USD per question (Amendment 2, B2: uncached input 0.10, cached 0.01, cache
write 0.125, output 0.50 per 1M tokens; every request of a run, helpers included), averaged over
the 3 repeats. B\* = the published-baseline cell with the highest repeat-averaged **test** F1
(ties: lower cost, then name), selected on test exactly as in v2. Paired cluster bootstrap over
test questions, stratified by KB block, 2,000 replicates, seed 7, percentile 95% CIs; repeats
are averaged within a question before resampling; ratios are ratios of means.

**H8 supported** iff all of:
- (a) cost(aob) / cost(B\*) <= 0.70 as a point estimate AND its 95% CI upper bound < 1.0;
- (b) non-inferiority in F1: F1(aob) - F1(B\*) >= -2.0 points as a point estimate (inclusive)
  AND its 95% CI lower bound >= -5.0 points (inclusive);
- (c) frontier: no published baseline or reference cell (rag1, closed_book) has both strictly
  lower mean cost and strictly higher mean F1 than aob;
- (d) for every published baseline cell b with F1(b) >= F1(aob) - 2.0 points,
  cost(aob) / cost(b) <= 0.70 with its 95% CI upper bound < 1.0.

Otherwise not supported, reported as is; an undefined CI = not supported (unstable). H8 is "not
yet run" if aob or every published cell is missing. **Registered cache-isolation check:**
provable cross-run cached tokens (cached tokens of a request whose own run attempt had no
earlier request of at least 1,024 prompt tokens) must total 0 on test; if not, the primary cost
becomes the cache-neutral USD below, and this is reported.

Secondary, all descriptive and reported whichever way they fall, each with its 95% CI:
- **Per-arm table** (every cell, losing arms included): billed USD per question including
  cache-write premiums; cache-neutral USD (every input token at 0.10) and no-write-premium USD
  (writes at 0.10, reads at 0.01) from the same logged usage; tokens by kind (input, cached,
  cache write, output, reasoning); cache hit rate (cached / input); requests; turns; latency;
  EM and F1 with CIs; paired differences to aob (F1, EM, cost ratio) with CIs.
- Variance across repeats per cell: SD and range of the per-repeat means of F1, EM and cost,
  and the share of questions whose normalised answer is identical in all repeats.
- H8 recomputed on cache-neutral and on no-write-premium USD.
- Ablations: aob against each ablation (cost ratio and F1 difference).
- aob against rag1, closed_book and pbc.
- Gate statistics: answer requests per run, share committed at requests 1/2/3, supported share,
  F1 by supported flag, share of aob requests at or above 1,024 prompt tokens, median prompt.
- The billed and cache-neutral frontiers; the cross-run check per arm.
- The winner's curse in B\* (a maximum over near-identical loop cells, which works against aob)
  and temperature-0 nondeterminism are stated with the results.

## 9. Budget and the [REDACTED: unrelated]

`MAX_USD` stays 5 for the project; spent before v3: 0.041781 USD (smoke 0.000243 and v2 dev
round 0, 0.041538). **v3 spend stop: 3.00 USD**, enforced as a project total of 3.04 USD
(`budget.max_usd` in the v3 configs; the grid takes the minimum of it and any dispatch input).
Estimated at the dev-round-0-calibrated rate of about 0.0024 USD per question for all 13 cells:
dev round 0 about 7,900 requests and 0.70 USD; dev rounds 1-2 about 400 requests and 0.05 USD
each; test about 19,700 requests and 1.75 USD. [REDACTED: unrelated]
[REDACTED: unrelated]: the Amendment 2 B7 rules hold (per-request worst-case admission, concurrency at most
2, bounded backoff on 429 and 5xx, an immediate whole-run halt on `insufficient_quota` or a
billing stop, no further request until the author confirms). The key is never printed, logged or
committed.

## 10. Supersession of v2 and deviations from it

v2 is superseded after its dev round 0. Its dev table and the `pbc` result (dominated; cost
ratio 1.12, CI 0.96-1.34; F1 -4.3 points, CI -13.8 to +1.9) stay in DECISIONS.md (D31) and are
printed in the paper as superseded, never deleted. v2's dev costs are biased by the cross-run
cache confound (D33), which v3 removes. H7 was never evaluated. Harness changes from v2, all
applied to every arm: H1-H3 and H5-H6; notes removed from react, window2 and memgpt (their
sources have none); react's cap of 7; agentless without cites or ids; 48-question KB blocks;
new splits. v3 was designed after seeing v2 dev data, so only v3 test results are confirmatory.

## 11. Threats stated now

One model and one provider; one task family; BM25 retrieval; runs of about two requests (the
superlinear term is not tested); possible memorisation of HotpotQA by the model (closed_book
measures it, and it affects every arm); temperature-0 nondeterminism (3 repeats); the winner's
curse in B\*; the venue (a retrieval QA task as a controlled test of an agent cost mechanism).

## Appendix A. Exact prompt strings

The salt message is `session <16 hex>`. Loop arms use the v1 layout (system prompt, user
`Question: {q}`, then assistant reply / user `Observation: {obs}` pairs as their policy keeps
them). rag1 user message: `Question: {q}` + blank line + `Passages:` + newline + lines + blank
line + reader instruction. agentless request 1: the same with the agentless query instruction;
request 2: `Passages (question search):` and `Passages (follow-up search):` sections, then the
reader instruction. closed_book: `Question: {q}` + blank line + its instruction. Passages are
`Title: text` lines without ids for every non-loop arm.

**ANSWER_CONTRACT (H3; byte-identical in every v3 arm)**

```text
The answer must be as short as possible: yes or no, or the shortest span that answers the question, copied verbatim from the passages when they contain it.
```

**react and window2 system prompt (`react_v3_prompt(7)` for react, `react_v3_prompt(10)` for window2 and memgpt)**

```text
You are a careful research assistant answering one question with a small tool set.

Tools:
- search(query) -> returns the top-k passages relevant to the query, each line prefixed with [doc_id].
- read(doc_id) -> returns the full paragraph of one document.
- finish(answer) -> ends the run; answer is a short span of text or yes/no.

Reply with exactly one JSON object of the form {"thought": ..., "action": ..., "args": ...}:
- "thought": brief reasoning about the next step.
- "action": one of "search", "read", "finish".
- "args": for search {"query": ...}, for read {"doc_id": ...}, for finish {"answer": ...}.

Rules:
- Take at most 7 steps.
- The answer must be as short as possible: yes or no, or the shortest span that answers the question, copied verbatim from the passages when they contain it.
- When you have enough information, call finish.
```

**notes_reset system prompt (`notes_v3_prompt(10)`)**

```text
You are a careful research assistant answering one question with a small tool set.

Tools:
- search(query) -> returns the top-k passages relevant to the query, each line prefixed with [doc_id].
- read(doc_id) -> returns the full paragraph of one document.
- finish(answer) -> ends the run; answer is a short span of text or yes/no.

Reply with exactly one JSON object of the form {"thought": ..., "notes": ..., "action": ..., "args": ...}:
- "thought": brief reasoning about the next step.
- "notes": a running summary of all facts found so far that matter for the question, at most 60 words. Write it as if earlier observations may become invisible.
- "action": one of "search", "read", "finish".
- "args": for search {"query": ...}, for read {"doc_id": ...}, for finish {"answer": ...}.

Rules:
- Take at most 10 steps.
- Keep the notes complete: earlier observations may no longer be visible.
- The answer must be as short as possible: yes or no, or the shortest span that answers the question, copied verbatim from the passages when they contain it.
- When you have enough information, call finish.
```

**Parse-error observation of the notes-free arms (REACT_V3_ERROR; notes_reset keeps the v1 text)**

```text
Error: the reply must be exactly one JSON object with keys "thought", "action", "args" and "action" one of "search", "read", "finish". This step counts; recover with a valid action.
```

**memgpt summariser prompt (SUMMARIZER_PROMPT, unchanged from v2)**

```text
You compress the earlier steps of a research agent into a short factual summary. Keep every fact, name, number and doc_id that could matter for answering the question; drop everything else. Reply with exactly one JSON object {"summary": "..."} of at most 150 words.
```

**Reader system prompt (agentless, rag1)**

```text
You are a careful research assistant answering one question from retrieved passages. Reply with exactly one JSON object and nothing else.
```

**Reader answer instruction (agentless request 2, rag1)**

```text
Answer the question using the passages. The answer must be as short as possible: yes or no, or the shortest span that answers the question, copied verbatim from the passages when they contain it. Reply as {"answer": "..."}.
```

**agentless request 1 instruction (AGENTLESS_QUERY_INSTRUCTION, unchanged from v2)**

```text
Write one follow-up search query that would find the missing evidence needed to answer the question. Reply as {"query": "..."}.
```

**closed_book system prompt**

```text
You are a careful research assistant answering one question. Reply with exactly one JSON object and nothing else.
```

**closed_book answer instruction**

```text
Answer the question. The answer must be as short as possible: yes or no, or the shortest span that answers the question, copied verbatim from the passages when they contain it. Reply as {"answer": "..."}.
```

**AOB_PROMPT (aob and every ablation)**

```text
You answer one question from the passages given. Reply with exactly one JSON object {"answer": ..., "next": [...]}:
- "answer": your best answer, even if unsure. The answer must be as short as possible: yes or no, or the shortest span that answers the question, copied verbatim from the passages when they contain it.
- "next": one or two short search queries for the fact your answer depends on most; they are used only if the passages do not state your answer.
```

**AOB_RETRY_LINE (`{answer}` = the previous answer)**

```text
Your previous answer "{answer}" is not stated verbatim in the passages; new passages were added.
```

**aob_llmq query request: system = closed_book system prompt; user = `Question: {q}` + blank line + this instruction**

```text
Write one search query that would find the evidence needed to answer the question. Reply as {"query": "..."}.
```

**AOB_THOUGHT_PROMPT (registered dev-round-1 variant; used only if that round is triggered)**

```text
You answer one question from the passages given. Reply with exactly one JSON object {"thought": ..., "answer": ..., "next": [...]}:
- "thought": brief reasoning, at most 20 words.
- "answer": your best answer, even if unsure. The answer must be as short as possible: yes or no, or the shortest span that answers the question, copied verbatim from the passages when they contain it.
- "next": one or two short search queries for the fact your answer depends on most; they are used only if the passages do not state your answer.
```

**pbc system prompt (PBC_V3_PROMPT: the v2 prompt with the answer contract)**

```text
You are a careful research assistant answering one question with a small tool set.

Tools:
- search(query) -> returns the top passages relevant to the query, each line prefixed with [chunk_id] (of the form doc_id#n).
- read(doc_id) -> returns every passage of one document, each prefixed with its [chunk_id].
- finish(answer) -> ends the run; answer is a short span of text or yes/no.

Reply with exactly one JSON object of the form {"thought": ..., "notes": ..., "action": ..., "args": ..., "cites": ...}:
- "thought": brief reasoning about the next step.
- "notes": a running summary of all facts found so far that matter for the question, at most 60 words. Write it as if earlier observations may become invisible.
- "action": one of "search", "read", "finish".
- "args": for search {"query": ...}, for read {"doc_id": ...}, for finish {"answer": ...}.
- "cites": a list of the chunk_ids (from the latest observation or the Evidence block) that you rely on right now, [] if none.

Context rules:
- You see only the Evidence block (passages you cited earlier), your notes, and the latest action and observation. Every other passage is dropped after one viewing.
- Cite every passage you will need again; cited passages are kept verbatim in the Evidence block.
- A finish must cite at least one chunk_id of a passage you retrieved; an uncited answer is sent back once.
- Take at most 10 steps.
- The answer must be as short as possible: yes or no, or the shortest span that answers the question, copied verbatim from the passages when they contain it.
- When you have enough information, call finish.
```

## Appendix B. Fail-first evidence

Every new test was shown to fail on broken code before it passed: the harness tests against the
v2 grid runner, the split test with a test block moved onto a touched range, and the AoB and
analysis tests by single mutations, each restored afterwards. The outputs are in
`experiments/offline/failfirst_v3.txt`.

## Amendment 1 (2026-11-16, before any v3 request; operational fixes from an independent review)

Registered by the commit that adds this section. No v3 request of any kind (smoke included) had
been made when it was written. An independent read-only review of the registration commit
(dc9634d) found no defect in the salt, AoB or split code and six operational issues, fixed here.
Nothing in sections 1-8 that defines the arms, AoB, the splits, H8 or its thresholds changes.

C1. **Block estimate.** The block pre-check priced every run at 3,000 input and 300 output tokens
at the cache-write rate, which would refuse a test dispatch that the real cost (about 0.00017 USD
per run) fits easily. It now uses 1,500 and 150 (v2 dev round 0 measured 1,415 and 143 per run),
still priced at the cache-write rate, and counts only the (cell, question, repeat) runs not yet
done. The per-request worst-case guard (B7) is unchanged and remains the binding limit.
C2. **Halt across shards.** A billing stop (`insufficient_quota`, billing hard limit, inactive
billing, terminated access) in either shard writes the halt file `out/h2h3_HALT`; while it exists
no request is sent by either shard or by any later dispatch that resumes the chain (the file
travels with the artifact). Only the author clears it, by a dispatch with `models=clear-halt`.
C3. **`questions` is a total** over the split (the first N questions), applied before the two
shards split it; the shard partition is unchanged across resumed dispatches.
C4. **Inclusive thresholds are float-safe**: H8 (b)'s -2.0 and -5.0 points and rule (d)'s
-2.0 points are compared with a tolerance of 1e-9 (and rule (a)'s 0.70 likewise); the strict
comparisons (CI upper bound < 1.0, rule (c), the T1 trigger) stay strict.
C5. **A retried run gets its own salt** (attempt number in the hash), so a retry cannot read its
failed attempt's cache; the salt actually sent is recorded per request.
C6. **B\* eligibility needs completeness**: a published cell is B\*-eligible only if it has runs on
at least 95% of the questions any cell answered; cell EM and F1 are the means over questions of
the repeat-averaged per-question values (as section 8 specifies for the paired statistics).
A cell stopped by 3 consecutive errors records every remaining repeat as NOT RUN.

Correction to section 9: the grid has **15** cells (7 published, 2 references, aob, 4 ablations,
pbc), not 13; the request and USD estimates there already counted all 15 (about 27 requests and
0.0025 USD per question per repeat). Test dispatch plan: five dispatches of one 48-question block
each (`questions` = 48, 96, 144, 192, 240), each resuming the previous artifact, about 75 minutes
each at concurrency 2.

## Amendment 2 (2026-11-16, after a failed smoke attempt, before any billed v3 request or data)

The first smoke attempt (Actions run 43805900879, 04:45Z UTC, commit 8d8bc79) made one request,
which the API rejected with HTTP 401 `token_invalidated` ("Your API key has been invalidated").
Nothing was billed (spend stays 0.041781 USD) and no reply or data exists. The run recorded it
as an ordinary error and the job still ended green. From now on an HTTP 401 (a missing, invalid
or revoked key) is handled like a billing stop (C2 of Amendment 1): never retried, the whole run
halts, the halt file is written and the job fails. Nothing else changes. The smoke (section 6.1)
is repeated once a valid key is in the `OPENAI_API_KEY` secret.

## Amendment 3 (2026-11-16, credential change and history rewrite; before any further request)

Registered by the commit that adds this section, before any request after the failed smoke
attempt of Amendment 2. **No design change.**

D1. **Credential.** On the author's instruction, the Actions secret `OPENAI_API_KEY` was reset
at 2026-11-16T05:13Z UTC. It now holds a different key on the same account type
(the earlier key was invalidated, Amendment 2). The provider, the model `gpt-6-luna`, the
endpoint, every request parameter and the price sheet (Amendment 2 of PREREG_v2, B1-B5) are
unchanged. The project spend carries over (0.041781 USD so far), and the v3 spend stop stays at
3.00 USD (project total 3.04). Amendment 2's halt on HTTP 401 stays. The key is never printed,
logged or committed.

D2. **History.** On the author's instruction, `main` was rewritten to remove AI co-author
trailers. The trees are identical, and the new head is 5baf009. The work was rebased onto it,
contents unchanged, and is published as the new branch **exp/agent-rag-v3b**. The old branch
exp/agent-rag-v3 was not overwritten: a force-push was not permitted in this session. Its commits got new SHAs, and their author dates are kept:
- registration dc9634d -> **bfcde41** (authored 2026-11-16T04:31:15Z, first pushed
  04:31:20Z UTC);
- Amendment 1 8d8bc79 -> 9e0121d;
- Amendment 2 85a5d64 -> 2b29b46;
- fb1b3e0 and f203c81 replace 15db707 and f669c10.

The original SHAs stay in the Actions logs: CI run 43804612766 at dc9634d, and the smoke attempt,
run 43805900879, at 8d8bc79.

## Amendment 4 (2026-11-16, operational only; after the freeze e274b76, before any test request)

Registered by the commit that adds this section. An independent experiment audit of the freeze
e274b76 found no method problem; it reproduced D41 and D43 exactly. It asked for these fixes to
the orchestration before the test. **No design change:** `agent/arms.py`, `agent/agent.py`,
`agent/prompts.py`, `agent/data.py`, `experiments/configs/h2h_v3.yaml` and
`experiments/analyze_v3.py` are byte-identical to e274b76 (D44 lists the blob hashes), so the run
configuration hash (sha256 a6384cc6...) is unchanged.

E1. Each finished run is appended to `runs.jsonl` and flushed to disk at once, not only when the
grid ends. Duplicate records of a run, such as an error later superseded by its retry's ok
record, collapse to the latest when the file is read.
E2. On any exit of the run step, including a failure, timeout or cancel, the container is stopped
and its `out/` copied back before the artifact upload: a trap in `run_in_container.sh` and an
`always()` step in `experiment.yml` before the upload and the cleanup. The run step's own timeout
(165 minutes) is below the job's (180).
E3. An h2h3 job refuses to resume from a hollow artifact: one without `ci_context.txt`, or whose
dispatch left fewer ok runs than it was asked for (`experiments/ci/check_prev_v3.py`), not only
when `prev/` is missing.
E4. A spend-guard charge that has no request record (an attempt that may have been billed before
the request failed) is written as an `unrecorded_charge` record, so later jobs count it in prior
spend. `experiments/attempts_v3.py` reports the failed and unconfirmed attempts per arm and what
they cost.

Appendix A predates D42: the frozen `AOB_PROMPT` and `AOB_THOUGHT_PROMPT` read "always one or two
short search queries" (DECISIONS.md D42). The test procedure rules C1-C4 are in DECISIONS.md D45.

## Amendment 5 (2026-11-16, budget only; during the test, before dispatch 9's first request)

Registered by the commit that adds this section. It was corrected, docs only, before any request
under it, after the auditor re-checked c666e81.

**Change.** The v3 spend stop goes from a project total of 3.04 USD to 3.50 USD, still inside the
project's 5.00 USD cap (`budget.max_usd` in `experiments/configs/h2h_v3.yaml`, the only change to
a file that runs).

**Basis: cost trend only.**
- Each 24-question test dispatch has cost about 0.19-0.24 USD, including the worst-case charges
  for timed-out (unconfirmed) attempts.
- The block pre-check estimates 0.14175 USD per shard (540 remaining runs x 0.0002625 USD), that
  is 0.2835 USD for both shards together, against half of the remaining budget per shard. Under
  3.04 USD it refuses a dispatch once prior spend exceeds 3.04 - 0.2835 = 2.7565 USD.
- With 1.859 USD spent after dispatch 5, dispatch 10 would be refused if dispatches 6-9 average
  about 0.225 USD or more, and dispatch 9 only if dispatches 6-8 average about 0.30 USD or more.
  At the observed rate, dispatch 10 was at risk.

**Author's options (relayed by the orchestrator):** (a) raise the stop to 3.50 USD; (b) keep
3.04 USD and stop at the n reached (D45 C3). The author chose (a), to complete the registered
n = 240. Interim H8 printouts existed in the job logs and were **not consulted**.

**Supersedes D45 C3's fallback.** For this test, Amendment 5 supersedes the fallback "report at the
n reached". That fallback was written for an unforeseen budget refusal. This refusal was
foreseen from spend alone and resolved by the author before it happened, so the test continues to
the registered n.

**Unchanged:** arms, prompts, splits, H8 and the analysis.

**Mechanism.** Dispatches 9 and 10 run from the tag `prereg-v3-freeze3`. Its runnable files differ
from `prereg-v3-freeze2` (f6bb418) only in this one constant; DECISIONS.md D49 lists the files and
blob hashes.

**Permission scope.** The author's permission for the key covers the remaining test dispatches and
any single D47 re-run, up to the new total.
