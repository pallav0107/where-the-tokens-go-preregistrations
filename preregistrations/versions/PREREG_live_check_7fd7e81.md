Status: REGISTERED — 17 Nov 2026 (UTC). Committed before any data was collected.

# PREREG GATEWAY-LIVE: idle gaps, cache lifetime and re-processing on a local-model harness

Drafted and registered on 17 Nov 2026 between 12:15 and 12:59 UTC.
No experimental session described here has been run. Section 9 lists the feasibility probes
made before writing this file. They were API capability checks, and none of their numbers
enter the analysis.

**Code freeze.** The runner (`algorithm/benchmarks/gateway_live/run_live.py`), the analyzer and
their tests are committed after this file. Before the first session, this file is re-committed
with an amendment line naming the frozen code commit. From then on,
`algorithm/benchmarks/prereg_gate.py` (code path `algorithm/benchmarks/gateway_live`) refuses
to run if that code has changed since this file's last commit.

## 1. Purpose

The AGENT 2027 paper (`papers/agent2027-cost/`) measures a Claude Code harness. Input there is
dominated by re-sent cached context, and most cache writes re-write context that expired
during idle gaps longer than the cache lifetime. The paper's second harness is an open-source
agent gateway, called "the gateway" below and named in the internal records. It now runs only
local models, so nothing is billed. The same mechanism should cost GPU time and latency instead:
after an idle gap longer than the model server's lifetime, the next request re-processes the
whole prompt.

This experiment checks that mechanism live, on the model servers the gateway uses. It is a
mechanism check: it measures neither the gateway's historical usage nor any bill.

## 2. Setup (verified 17 Nov 2026; the runner records versions and settings per run)

| Arm | Server | Model | Lifetime that matters |
|---|---|---|---|
| `box-ollama` | Ollama 0.30.10 on the box GPU (RTX PRO 4000 Blackwell, 24 GB), `127.0.0.1:11434`, the endpoint the gateway's local provider calls | `agentic:32k`: the same weights blob (qwen35, 27.8B, Q4_K_M) as `agentic:latest`, which the gateway's agents use, with a 32,768-token context | keep-alive 5 min (`OLLAMA_KEEP_ALIVE` unset, so the default; the probe saw `expires_at` = last response + 5 min) |
| `spark-ollama` | Ollama 0.32.15 on the DGX Spark (GB10), port 11434 on the box-Spark private link | `oclive-qwen35-27b:32k`: the same weights blob, copied from the box, with the same parameters | keep-alive 30 min (`OLLAMA_KEEP_ALIVE=30m` in its unit) |
| `spark-vllm` | vLLM (image `vllm/vllm-openai:nightly-aarch64`, version recorded) on the Spark, bound to the private-link address only (port 8010), prefix caching on, `--enable-prompt-tokens-details`, `gpu_memory_utilization` 0.30 | `Qwen/Qwen2.5-7B-Instruct` (bf16, already in the Spark's HF cache, no vision tower) | no time limit; the prefix cache is memory-bounded (LRU) |

Why not the full `agentic:latest`: its 262,144-token context does not fit the 24 GB GPU. The
probe placed 22.8 of 28.4 GB on the GPU, spilled the rest to system RAM, and left 0.6 GB of
GPU headroom. The author's limit is 1-2 GB, and a split model would time the CPU rather than
the cache. `agentic:32k` uses the identical blob and runs fully on the GPU, with about 5.7 GB of headroom.

**Not driven through the gateway, and why.** The gateway's `agent --session-id` command is
scriptable, but the gateway calls Ollama's OpenAI-compatible endpoint. That endpoint returns
only `usage.prompt_tokens` and drops every timing this experiment needs: prompt-evaluation
time, load time and time to first token. The runner therefore calls the same endpoints with
the same weights. It uses Ollama's native `/api/chat` and vLLM's `/v1/chat/completions`. It
sends no `keep_alive`, `num_ctx` or cache option, so the server defaults apply exactly as they do for
the gateway's calls. No gateway agent, session, workspace or configuration is touched, and the
experiment's model names and session nonces start with `oclive`. Endpoint addresses are read
from a box-local file outside git (`endpoints.json` in the run directory).

## 3. Task (identical in every arm and session)

* **Context.** A fixed system message tells the model it is a code-review agent. It starts with
  a fresh random session nonce, so no session can reuse another's prefix, and then holds about
  6,000 tokens of source code. The code comes from fixed Python standard-library modules of
  the runner's interpreter, and the runner records the interpreter version and a sha256 of the text.
* **Turns.** Each user turn asks for a review of the next roughly 400-token section of a second
  fixed module. The model's reply is appended and the context grows by appending only. Output
  is capped at 96 tokens, with temperature 0, seed 7 and thinking off (Ollama `think: false`).
* **Streaming.** On for every request, so time to first token is measured client-side.
* **Content.** The repository never holds any prompt or response text. Raw per-request records
  are numbers only and stay on the box in the run directory ([REDACTED: infrastructure]).

## 4. Experiments

**E1. Idle gap.** A session runs K = 6 turns. It then sleeps for the gap g, measured from the
end of turn 6's response to the start of the next request. Turn 7 is the measured request M,
and turn 8 follows immediately as a post-gap control. The gaps are g ∈ {0, 2, 4, 6, 10} min,
bracketing the box's 5 min lifetime. Each arm runs 3 blocks, and each block holds every gap
once, in an order shuffled with `random.Random("<arm>-block<b>")`. Sessions run back to back
on one arm at a time. A new session never reuses a prefix because of its nonce.

**E2. Context growth.** On each arm, 3 sessions of N = 16 turns run with no gap.

**E3. Keepalive, box only.** The condition `10+ping` runs inside each `box-ollama` block. A
10 min gap gets pings at 4 and 8 min, before the 5 min lifetime runs out. Each ping re-sends
turn 6's request with a 1-token output.

## 5. Measurements, per request (numbers only)

| Metric | Ollama arms | vLLM arm |
|---|---|---|
| prompt tokens | `prompt_eval_count`. In this version it counts all prompt tokens, cached ones included (probe, §9) | `usage.prompt_tokens` |
| reused tokens | not reported by Ollama. Estimated re-evaluated tokens = min(prompt tokens, `prompt_eval_duration` × r), where r is the same session's turn-1 cold throughput (tokens/s); reused = prompt − re-evaluated, labelled **estimated** | `usage.prompt_tokens_details.cached_tokens`, exact |
| prompt-processing time | `prompt_eval_duration` | not reported per request, so time to first token stands in |
| model load time | `load_duration` | none |
| time to first token, total latency | client clock, send to first non-empty streamed token, and to the final chunk | same |
| mechanism check before M | `/api/ps`: whether the arm's model is loaded, and its `expires_at` | `/metrics`: `vllm:prompt_tokens_total` before and after the gap (detects foreign requests) |
| contention | box: `nvidia-smi` GPU processes and memory at session start and end; Spark: `MemAvailable` | same |

## 6. Hypotheses and decision rules (fixed now)

"Within lifetime" means g ∈ {0, 2, 4}. "Beyond lifetime" means g ∈ {6, 10}; for `box-ollama`
that is past 5 min, and for the Spark arms it is the same gaps, which are still inside their
lifetimes. PP is the prompt-processing time at M.

* **H1 (box-ollama, the mechanism).** PP jumps once the gap exceeds the lifetime. SUPPORTED
  only if all three hold:
  1. the beyond-lifetime median PP is at least 3 times the within-lifetime median PP;
  2. a one-sided exact permutation test on PP gives p < 0.05 (all C(15,6) relabellings);
  3. the model was unloaded before M in at least 5 of 6 beyond-lifetime sessions, and loaded
     in at least 8 of 9 within-lifetime sessions.

  Time to first token, load time and total latency are reported in the same way.
* **H2 (spark-ollama, lifetime control).** No jump within a 30 min lifetime. SUPPORTED if the
  beyond-gap median PP is at most 2 times the within-gap median PP, and the model was loaded
  before M in every session.
* **H3 (spark-vllm, memory-bounded cache).** No expiry with idle time. SUPPORTED if the median
  of cached_tokens / prompt_tokens at M is at least 0.80 in both gap groups.
* **H4 (E2, context × requests).** Prompt tokens rise with the turn index. SUPPORTED if the OLS
  slope of prompt tokens on turn is positive in every session of every arm. The median reused
  share over turns 2-16 is reported as exact for vLLM, where the prediction is at least 0.80,
  and as estimated for Ollama. Also reported: the input multiple, Σ prompt tokens / final
  prompt tokens, and the re-sent share, Σ_{t≥2} prompt_{t−1} / Σ_{t≥2} prompt_t.
* **H5 (E3, keepalive).** SUPPORTED if all three hold:
  1. the model was loaded before M in at least 2 of 3 ping sessions;
  2. the median PP with pings is at most 2 times the median PP at g = 0;
  3. the net GPU time saved is positive. Saved = median (PP + load) at 10 min without pings,
     minus median (PP + load) at 10 min with pings, minus the median summed server time of
     the pings.

Every hypothesis is reported, whatever its outcome. A failed decision rule is reported as NOT
SUPPORTED and is never re-thresholded.

## 7. Analysis

`algorithm/benchmarks/gateway_live/analyze.py` reads the raw records. It writes
`evidence/gateway_live.json`, with aggregates and figures in the evidence envelope, so
`tools/gen_macros.py` makes the paper's macros. For each arm and gap it reports n, the median,
the minimum and the maximum of every metric in §5, together with each decision rule's inputs
and its verdict. Medians are used because n is small, and there are no further tests beyond
those in §6. Unit tests on synthetic fixtures pin the computations, and each test was shown to
fail before the code that passes it existed.

## 8. Exclusions, stopping rule and resource limits

* **Contaminated sessions.** A session is contaminated and excluded from the confirmatory
  analysis if any of these holds, and every flagged session is counted and reported:
  - on Ollama with g below the lifetime, the model is absent before M, or its `expires_at`
    differs by more than 20 s from (end of turn 6 + lifetime), meaning another client used it;
  - on Ollama with g above the lifetime, the model is present before M (another client
    re-loaded it);
  - on vLLM, `prompt_tokens_total` grew during the gap;
  - on the box, a GPU compute process other than Ollama and the [REDACTED: unrelated] server was present at
    session start or end.

  Each arm re-runs its contaminated conditions at the end of the arm, at most twice per condition.
* **Stopping rule.** Each arm stops after its 3 E1 blocks, its 3 E2 sessions and any re-runs.
  The design is fixed, so there is no optional stopping on results. If the arms are not
  complete by Wed 19 Nov 2026 07:00 UTC, only completed blocks are analysed and the gap is
  reported.
* **Resource limits (author).**
  - Box GPU: never touch the [REDACTED: unrelated] server, and keep at least 1.5 GB of GPU memory free.
  - Spark: before loading anything, `MemAvailable` must cover the planned reservation plus
    25 GB. Sessions are abandoned if `MemAvailable` falls below 30 GB.
  - Spark arms run one at a time. The vLLM container and the experiment's Ollama model are
    stopped or unloaded when their arm ends.
  - CPU work runs under nice/ionice, and no paid API is used.

## 9. Feasibility probes made before registration (not data)

On 17 Nov 2026 at about 12:13 UTC, three requests went to each of `agentic:latest` and
`agentic:32k` on the box, before this design was fixed: a cold request on about 3.2k tokens,
an exact repeat, and an append of about 0.4k tokens. They established five facts:
1. `prompt_eval_count` counts cached tokens. It read the same on the repeat.
2. `prompt_eval_duration` falls when the prefix is reused: on `agentic:32k` it went from 4.1 s
   cold, to 0.09 s on the repeat, to 0.67 s on the append. So prefix reuse works on this hybrid
   (recurrent plus attention) architecture.
3. The box keep-alive is 5 min.
4. `agentic:latest` spills to CPU; `agentic:32k` does not.
5. The response carries no cached-token field.

These numbers motivated the 3x threshold in H1, so H1 is a mechanism check, not a test of a
surprising prediction. The Spark was inspected but no model was loaded there.

## 10. Use in the paper

At most 4-5 sentences go next to the second-harness paragraph. Every number is a macro from
`evidence/gateway_live.json`, and the sentences are labelled pre-registered, with any
deviation stated. A null or messy outcome is reported as such.

## 11. Amendment 1 (17 Nov 2026, before any experimental session)

* **Code freeze.** The runner is frozen at commit `ffe9e62`. The gate's code path is the runner
  file `algorithm/benchmarks/gateway_live/run_live.py`, not the whole directory. The analyzer
  is therefore committed separately, implementing sections 6-7 as written, before any result
  is read. `spark_vllm.sh` (start and stop the vLLM server, with the memory check of section 8)
  is not part of the frozen runner.
* **Permutation statistic.** H1's exact permutation test uses the difference in means (beyond
  minus within). Section 6 named the test but not its statistic.
* **Smoke tests (not data, never analysed).** These checked the runner against the real
  servers: one gap-0 session each on `box-ollama` and `spark-ollama` (about 12:31-12:34 UTC),
  and one on `spark-vllm` before that arm's confirmatory run. They wrote to a separate `smoke/`
  directory and changed nothing in the design.
* **Spark model identity.** The Spark's `oclive-qwen35-27b:32k` was created from the copied
  blob. Its weights digest equals the box's (`sha256:83c54730…`), with the same template,
  renderer, parser and parameters.
* **Clock offset.** The box and Spark clocks differ by about 2.7 s (seen in `expires_delta_s`),
  well inside the 20 s tolerance of section 8.
* **Gaps.** The 4 min gap leaves the box's 5 min lifetime with about 1 min of margin, and the
  6 min gap passes it by about 1 min, timed from the end of turn 6. This is unchanged and
  stated for clarity.


## 12. Amendment 2 (17 Nov 2026, about 13:15 UTC: sessions running, no measured request or check record read)

This amendment follows an independent methodology audit, made before any result was read. It
moves no threshold.

1. **Manipulation checks before exclusion.** Section 8's exclusion keys on the same
   `loaded_before` flag, so after exclusion these checks would be true by construction. They
   are therefore computed on all first-attempt E1 sessions (`rerun_of` empty), contaminated
   sessions included:
   - H1 rule (c);
   - H2's "loaded in every session";
   - H5 rule (a).

   PP comparisons stay post-exclusion, as registered. Contamination is counted by condition
   and by reason, and re-runs are counted too.
2. **Power.**
   - If any gap cell (0, 2, 4, 6, 10) has fewer than 2 clean sessions, H1, H2 and H3 are
     reported as UNDERPOWERED and not decided. H5 follows the same rule for cells 0, 10 and
     10+ping.
   - With fewer than the full set of sessions, the permutation test enumerates all
     relabellings of the sessions that exist. Rule (c) then needs all but one session per group.
3. **Which runs count.** For each arm, the earliest run whose manifest is confirmatory and
   carries a gate commit. Other runs are ignored and counted. If a run is aborted, its
   completed sessions count.
4. **Throughput r and the reuse estimate.**
   - r = turn-1 prompt tokens / turn-1 `prompt_eval_duration`. That duration excludes load time.
   - The Ollama reuse figure is labelled "estimated (timing-based, not server-reported; likely
     a lower bound on reuse)".
   - It is descriptive only. It never feeds a decision rule and is never pooled with vLLM's
     exact counts.
5. **Actual gap.** The actual gap is M's send time minus the end of turn 6 (the check record's
   `gap_actual_s` is stamped just before M is sent). Conditions keep their nominal labels.
6. **Registered sensitivity analyses.** These are descriptive and do not change the
   confirmatory verdicts.
   - (a) H1 recomputed without any exclusion, on all first attempts.
   - (b) The Ollama server journal (`ollama_posts.py`). For each clean E1 session, any chat
     POST from another client counts as foreign use. A chat POST is one to `/api/chat`,
     `/api/generate`, `/v1/chat/completions` or `/v1/completions`. It counts if its span
     overlaps the window from turn-6 end + 1 s to M send − 0.5 s, after subtracting the
     session's own pings.
     - H1 rules (a) and (b) are recomputed without the flagged sessions.
     - Embedding POSTs are counted and reported.
     - Spark journal times are shifted by a clock offset measured over ssh.
   - (c) A reload warm-up check: PP per 1,000 prompt tokens at M beyond the lifetime, compared
     with turn 1.
7. **Deviations and exposures, disclosed.**
   - `box-ollama` and `spark-vllm` ran at the same time from the same client box, on different
     GPUs. Server-side timings are unaffected. Client-measured TTFT and total latency shared
     the box's CPU and network.
   - Spark memory is checked at session start and end, not mid-session as section 8 says. A
     watcher alerts below 40 GB. The box GPU floor is checked at session start only.
   - The session nonces are random hex, not `oclive`-prefixed, and `agentic:32k` is a
     pre-existing model name.
   - Contaminated E2 sessions are not re-run.
   - Other clients of the box's Ollama are only partly detectable. The GPU check allows any
     Ollama process, and another client loaded an embedding model during the box arm. That is
     why 6(b) exists.
   - The 30-min Spark arm is a supporting negative control, not a one-variable manipulation of
     the lifetime. Apart from the lifetime, it differs from the box arm in:
     - hardware;
     - Ollama version;
     - server settings (flash attention and q8_0 KV cache on the box only);
     - network path;
     - time of run.

     The primary evidence is the within-arm contrast on the box.
   - Seen during collection, before this amendment:
     - the smoke tests' printed timings;
     - turns 1-6 (pre-gap requests only) of the first session of `box-ollama` and
       `spark-vllm`, checked to confirm the logging;
     - the watcher's session and contamination counts.

     No measured request and no check record had been read.
   - The analyzer was written after collection began: an executor drafted it and the author
     reviewed it. It is committed with this amendment, before any measured request is read,
     with tests that were shown to fail first.
