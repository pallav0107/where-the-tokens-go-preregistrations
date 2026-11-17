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
