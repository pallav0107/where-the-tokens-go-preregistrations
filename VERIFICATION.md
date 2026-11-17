# Verification of registration timing

All times are UTC. Each row's full record (commit, author date, the server event, the original
file's git blob hash and the released file's sha256) is in `MANIFEST.json` under
`registrations` and `actions_runs`.

## What an independent audit verified

An independent audit of the controlled experiment, separate from the author's own checks, is
recorded in the released decision log (`package_A/DECISIONS_excerpt.md`):

- **D44.** An independent audit verified the freeze `e274b76` before any test request. It found
  no method problem (verdict GO-WITH-CONDITIONS) and reproduced the dev results of D41 and D43
  exactly. Its conditions were operational and became Amendment 4. The files that define the
  design kept the same git blob hashes (listed in D44).
- **D47.** An independent re-audit verified the change from `e274b76` to `f6bb418`
  (GO-WITH-CONDITIONS) before test dispatch 1. Test dispatches 1-8 ran from `f6bb418`.
- **D49.** The auditor re-checked `c666e81` (Amendment 5) before any request under it. It found
  the one-line budget change clean and confirmed that runs under the two configs pool correctly.

The audit's own working material is not part of this release. That material is the downloaded
CI artifacts of every v3 run, copies of the frozen code trees and the audit's check scripts.

## Server-side timing, checked for this release

For each registration, the table gives the time GitHub first recorded the commit and the time
the first CI run under it was created. Both clocks are GitHub's: the repository activity log for
pushes and the Actions API for runs.

| Registration or amendment | Commit | First on GitHub | First CI run it governs (created) | Gap |
|---|---|---|---|---|
| Held-out test (PREREG_heldout) | dd10b40 | 15 Nov 12:33:35 | no CI run; the first results commit (d7bf97f) reached GitHub at 13:10:23 | 36 min |
| v1 registration | 1f63cd0 | 15 Nov 11:42:02 | never run (no experiment request was made) | n/a |
| v1 Amendment 1 | f48d079 | 15 Nov 11:53:11 | never run | n/a |
| v2 registration | 260aa00 | 15 Nov 13:53:25 | dev round 0, run 43776254575, 23:42:27 | 9 h 49 min |
| v2 Amendment 2 | 860556b | 15 Nov 23:14:03 | run 43776254575, 23:42:27 (the one smoke call, at 23:26:29 by the workstation clock, came after the push too) | 28 min |
| v3 registration | dc9634d, rewritten as bfcde41 | 16 Nov 04:31:20 | first v3 run, 43805900879 (at 8d8bc79), 04:44:36 | 13 min |
| v3 Amendment 1 | 8d8bc79, rewritten as 9e0121d | 16 Nov 04:40:09 | run 43805900879, 04:44:36 | 4 min 27 s |
| v3 Amendment 2 | 85a5d64, rewritten as 2b29b46 | 16 Nov 04:54:06 | next run that sent requests, 43809309111, 05:17:54 (run 43807166607 at 04:57:25 only ran the test suite, D46) | 24 min |
| v3 Amendment 3 | e859d0d | 16 Nov 05:16:53 | run 43809309111 (at e859d0d), 05:17:54 | 61 s |
| Freeze (tag `prereg-v3-freeze`) | e274b76 | 16 Nov 09:07:15 | none; the test plan, not a run | n/a |
| v3 Amendment 4 and re-freeze (`prereg-v3-freeze2`) | 925b525, f6bb418 | 16 Nov 09:49:44 and 09:50:17 | first test run, 43846158602 (at f6bb418), 10:08:42 | 18 min |
| v3 Amendment 5 | c666e81 | 16 Nov 20:18:27 | test dispatch 9, run 43967192969, 22:09:03 | 1 h 51 min |
| v3 Amendment 5 correction (`prereg-v3-freeze3`) | dcaa2e0 | 16 Nov 20:24:46 | run 43967192969 (at dcaa2e0), 22:09:03 | 1 h 44 min |
| Test results recorded | c936842 | 16 Nov 23:47:57 | after the last test run finished (run 43975815163, last updated 23:40:08) | n/a |

The test split (positions 4000-4239) was fixed in the v3 registration. It was first used by run
43846158602, about 5 h 37 min after the registration first reached GitHub.

## What these timestamps show, and what they do not

- **They show** that each registration and amendment was on GitHub's servers before GitHub
  created the CI run it governs. Each run records the commit it ran from (`head_sha`).
- **Pushes are upper bounds.** The activity log does not list tag creations, so a commit may
  have reached the server earlier than the time shown, but not later.
- **Author dates are set by the client.** The table does not rely on them.
- **The history rewrite.** The v3 registration and Amendments 1 and 2 exist under two SHAs each,
  because AI co-author trailers were removed from `main` (Amendment 3). The file blobs are
  identical at the old and the new SHA (`same_blob_as_rewritten_commit` in `MANIFEST.json`), and
  the original branch is still on the server. The table uses the original SHAs, which reached
  the server first. The rewritten SHAs arrived at 05:16:53, which is still before run
  43809309111.
- **Work done off the server leaves no server timestamp.** Three cases are disclosed in the
  documents themselves:
  - The held-out test's unit-building and pricing code was committed together with its first
    results, so git alone cannot show that this code predates them (`evidence/heldout.json`,
    notes).
  - v3's design-stage statistics ran locally, before the registration, on positions 54-1037.
    Those positions are listed and excluded from the test (PREREG_v3, section 2).
  - The v2 smoke call was sent from a workstation, so its time comes from that machine's clock.

## Live check (`PREREG_live_check.md`): server-side timing

Checked for this addition with the same method (activity log retrieved 2026-11-17T20:28:45Z; the
earliest push whose head contains the commit). The live check made no CI run: its sessions ran
from a local client against local model servers. The event each registration governs is therefore
the first confirmatory session, whose time comes from the runner's per-arm manifest (client clock,
recorded in the experiment's evidence file in the private repository), not from GitHub.

| Registration or amendment | Commit | First on GitHub | First session it governs (client clock) | Gap |
|---|---|---|---|---|
| Registration | 9df931c | 17 Nov 12:21:34 | first confirmatory session (arm `box-ollama`), 12:36:43 | 15 min 9 s |
| Amendment 1 (runner frozen at ffe9e62) | b1f5a8b | 17 Nov 12:36:25 | the same session, 12:36:43 | 18 s |
| Amendment 2 | 7fd7e81 | 17 Nov 13:05:03 | none; written during collection (see below) | n/a |
| Amendment 2 timestamp correction | 74f6576 | 17 Nov 13:05:17 | none; docs-only | n/a |

- **Sessions.** The arm manifests give `box-ollama` 12:36:43-14:34:19, `spark-vllm`
  12:40:05-14:02:57 and `spark-ollama` 14:03:19-15:32:48. Every manifest records b1f5a8b as both
  the runner commit and the gate commit (the commit that last changed the registration when the
  arm started). So Amendment 1 was in the runner's checkout before any confirmatory session,
  whatever the clocks say. The 18 s margin compares GitHub's clock with the client's, and holds
  only if the client clock was less than 18 s ahead of GitHub's.
- **Before the registration and Amendment 1.** The feasibility probes (about 12:13) and the smoke
  tests (about 12:31-12:34) are disclosed in the registration itself (sections 9 and 11). They
  are not data.
- **Amendment 2 came during collection**, as its own heading says: `box-ollama` and `spark-vllm`
  were running, and `spark-ollama` had not started. Server timestamps cannot show its statement
  that no measured request or check record had been read. They show only that it was on GitHub
  7 h 19 min before the analysis output: the evidence commit (015dd93) first reached GitHub at
  20:24:26, and its generator stamped 20:16:42 by the client clock.
- The caveats above apply here too: pushes are upper bounds, and author dates (listed in
  `MANIFEST.json`) are set by the client.
