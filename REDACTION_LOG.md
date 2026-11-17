# Redaction log (AGENT 2027 release)

Every line that was changed, with its category and the reason. The removed text is never shown here. Line numbers are those of the released file, which equal the original's (redaction keeps the line structure). Markers in the files: `[REDACTED: personal]`, `[REDACTED: unrelated]`, `[REDACTED: infrastructure]`.

## Totals (round 1)

Round 2 (personal, 58 rows), Round 3 (102 rows) and Round 4 (251 rows) are listed at the end.

| category | log rows | occurrences |
|---|---|---|
| personal | 0 | 0 |
| infrastructure | 111 | 112 |
| unrelated | 98 | 98 |
| **all** | 209 | 210 |

205 distinct lines changed; 4 of them carry two categories (one row each).

By package: package_A infrastructure 25; package_A unrelated 29; package_B infrastructure 86; package_B unrelated 69.

Round 1: no `personal` redaction was needed inside the released text: the only personal data found were the author byline (kept, as allowed) and identifiers inside CI hostnames and repository URLs, which are logged under `infrastructure` because the whole value was replaced. The `package_A/versions/` rows repeat the same edits in the earlier registered versions of the same files.

## What was dropped from the Package B records

- **JSONL records (runs, requests, requests_failed, not_run, tokenledger): nothing.** Every key of every record (73434 records) was checked against an allowlist of scientific and operational fields (question id, position, block, split, arm, policy, k, repeat, salt, run id, step/turn, tokens by kind, usage object, USD, worst-case USD, latency, answer, gold, reply, next queries, gate fields, EM/F1, status, reason, timestamps, request parameters). No key outside it exists, and no value holds a hostname, local path, runner name or account id, so the files are byte-identical to the archived originals (see `identical_to_source` in MANIFEST.json).
- **provenance.json (37 files: 9 shard directories + 28 per-dispatch copies):** `hostname` (the CI runner's machine name) and `github_run_url` (a private-repository URL with the account name) are replaced by visible markers. `github_run_id`, git SHAs, config and dataset hashes, salt nonces, salts, spend and timing fields are kept.
- **Not included at all:** the CI job logs (`*.log`), `ci_context.txt` and the CI copies of the evidence JSON inside each artifact (they repeat run URLs and runner details; the released evidence files are the committed ones), the harness transcripts, `evidence/raw/`, and the files listed in README.md under 'Left out'.

## Per file

### `package_A/DECISIONS_excerpt.md`

| line | category | reason |
|---|---|---|
| 67 | infrastructure | host machine name and a local directory path |
| 99 | infrastructure | name or label of the self-hosted CI runner machine |
| 100 | infrastructure | name or label of the self-hosted CI runner machine |
| 102 | infrastructure | name or label of the self-hosted CI runner machine |
| 105 | infrastructure | device name of the local inference machine (v1 design platform) |
| 113 | infrastructure | device name of the local inference machine (v1 design platform) |
| 117 | infrastructure | device name of the local inference machine (v1 design platform) |
| 120 | infrastructure | device name of the local inference machine (v1 design platform) |
| 135 | infrastructure | local configuration file path |
| 237 | unrelated | operational note: the API key was shared with another application |
| 238 | unrelated | operational note about another application that used the same key |
| 239 | unrelated | operational note about another application that used the same key |
| 294 | infrastructure | name or label of the self-hosted CI runner machine |
| 411 | infrastructure | name of a CI secret that belongs to another project |
| 411 | unrelated | reference to another project |
| 412 | unrelated | reference to another project |
| 625 | infrastructure | name of the CI host machine |

### `package_A/PREREG.md`

| line | category | reason |
|---|---|---|
| 54 | infrastructure | device name of the local inference machine (v1 design platform) |
| 154 | infrastructure | device name of the local inference machine (v1 design platform) |
| 274 | infrastructure | device name of the local inference machine (v1 design platform) |

### `package_A/PREREG_v2.md`

| line | category | reason |
|---|---|---|
| 185 | unrelated | operational note: the API key was shared with another application |
| 192 | unrelated | operational note: the API key was shared with another application |
| 193 | unrelated | operational note: the API key was shared with another application |
| 202 | infrastructure | name or label of the self-hosted CI runner machine |

### `package_A/PREREG_v3.md`

| line | category | reason |
|---|---|---|
| 164 | infrastructure | name or label of the self-hosted CI runner machine |
| 234 | unrelated | operational note: the API key was shared with another application |
| 241 | unrelated | operational note: the API key was shared with another application |
| 242 | unrelated | operational note: the API key was shared with another application |

### `package_A/versions/PREREG_1f63cd0.md`

| line | category | reason |
|---|---|---|
| 54 | infrastructure | device name of the local inference machine (v1 design platform) |
| 154 | infrastructure | device name of the local inference machine (v1 design platform) |

### `package_A/versions/PREREG_v3_2b29b46.md`

| line | category | reason |
|---|---|---|
| 164 | infrastructure | name or label of the self-hosted CI runner machine |
| 234 | unrelated | operational note: the API key was shared with another application |
| 241 | unrelated | operational note: the API key was shared with another application |
| 242 | unrelated | operational note: the API key was shared with another application |

### `package_A/versions/PREREG_v3_925b525.md`

| line | category | reason |
|---|---|---|
| 164 | infrastructure | name or label of the self-hosted CI runner machine |
| 234 | unrelated | operational note: the API key was shared with another application |
| 241 | unrelated | operational note: the API key was shared with another application |
| 242 | unrelated | operational note: the API key was shared with another application |

### `package_A/versions/PREREG_v3_9e0121d.md`

| line | category | reason |
|---|---|---|
| 164 | infrastructure | name or label of the self-hosted CI runner machine |
| 234 | unrelated | operational note: the API key was shared with another application |
| 241 | unrelated | operational note: the API key was shared with another application |
| 242 | unrelated | operational note: the API key was shared with another application |

### `package_A/versions/PREREG_v3_bfcde41.md`

| line | category | reason |
|---|---|---|
| 164 | infrastructure | name or label of the self-hosted CI runner machine |
| 234 | unrelated | operational note: the API key was shared with another application |
| 241 | unrelated | operational note: the API key was shared with another application |
| 242 | unrelated | operational note: the API key was shared with another application |

### `package_A/versions/PREREG_v3_c666e81.md`

| line | category | reason |
|---|---|---|
| 164 | infrastructure | name or label of the self-hosted CI runner machine |
| 234 | unrelated | operational note: the API key was shared with another application |
| 241 | unrelated | operational note: the API key was shared with another application |
| 242 | unrelated | operational note: the API key was shared with another application |

### `package_A/versions/PREREG_v3_e859d0d.md`

| line | category | reason |
|---|---|---|
| 164 | infrastructure | name or label of the self-hosted CI runner machine |
| 234 | unrelated | operational note: the API key was shared with another application |
| 241 | unrelated | operational note: the API key was shared with another application |
| 242 | unrelated | operational note: the API key was shared with another application |

### `package_B/code/agent/providers.py`

| line | category | reason |
|---|---|---|
| 26 | unrelated | operational note: the API key was shared with another application |
| 48 | unrelated | operational note: the API key was shared with another application |
| 49 | unrelated | operational note: the API key was shared with another application |

### `package_B/code/agent/tests/test_openai_provider.py`

| line | category | reason |
|---|---|---|
| 177 | unrelated | operational note: the API key was shared with another application |

### `package_B/code/experiments/offline/v2_dev_round0_run43777353380/agent_h2h_dev_preliminary.json`

| line | category | reason |
|---|---|---|
| 18 | infrastructure | private-repository CI URL (contains the account name); run id kept |

### `package_B/code/tools/pull_evidence.py`

| line | category | reason |
|---|---|---|
| 15 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 65 | infrastructure | home-relative local path to the harness scripts (workstation-only producer) |
| 793 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 794 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 795 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 796 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 797 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 798 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 799 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 800 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 801 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 802 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 803 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 804 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 805 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 806 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 807 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 808 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 809 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 809 | infrastructure | local file path |
| 810 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 810 | infrastructure | local file path |
| 811 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 812 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 813 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 814 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 815 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 816 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 817 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 818 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 819 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 820 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 821 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 822 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 823 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 824 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 825 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 826 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 827 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 828 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 829 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 830 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 831 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 832 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 833 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 834 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 835 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 836 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 837 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 838 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 839 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 840 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 841 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 842 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 843 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 844 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 845 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 846 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 847 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 847 | infrastructure | local file path |
| 848 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 849 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 850 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 851 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 852 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 853 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 854 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |
| 855 | unrelated | producer of an evidence file not in this release; it reads records of another project and of another article |

### `package_B/data/h2h3_dev_r0_s0/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/h2h3_dev_r0_s1/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/h2h3_dev_r1_s0/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/h2h3_dev_r1_s1/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/h2h3_smoke_8d8bc79/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/h2h3_smoke_r0/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/h2h3_test_r1_s0/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/h2h3_test_r1_s1/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/h2h_dev/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/provenance_by_dispatch/dev0-part1-run43810725916/h2h3_dev_r0_s0/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/provenance_by_dispatch/dev0-part1-run43810725916/h2h3_dev_r0_s1/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/provenance_by_dispatch/dev0-part2-run43821376770/h2h3_dev_r0_s0/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/provenance_by_dispatch/dev0-part2-run43821376770/h2h3_dev_r0_s1/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/provenance_by_dispatch/dev1-run43835072503/h2h3_dev_r1_s0/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/provenance_by_dispatch/dev1-run43835072503/h2h3_dev_r1_s1/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/provenance_by_dispatch/smoke-failed-run43805900879/h2h3_smoke_8d8bc79/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/provenance_by_dispatch/smoke-run43809309111/h2h3_smoke_r0/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/provenance_by_dispatch/test01-run43846158602/h2h3_test_r1_s0/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/provenance_by_dispatch/test01-run43846158602/h2h3_test_r1_s1/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/provenance_by_dispatch/test02-run43853944620/h2h3_test_r1_s0/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/provenance_by_dispatch/test02-run43853944620/h2h3_test_r1_s1/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/provenance_by_dispatch/test03-run43874434099/h2h3_test_r1_s0/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/provenance_by_dispatch/test03-run43874434099/h2h3_test_r1_s1/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/provenance_by_dispatch/test04-run43899808690/h2h3_test_r1_s0/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/provenance_by_dispatch/test04-run43899808690/h2h3_test_r1_s1/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/provenance_by_dispatch/test05-run43922204997/h2h3_test_r1_s0/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/provenance_by_dispatch/test05-run43922204997/h2h3_test_r1_s1/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/provenance_by_dispatch/test06-run43939566725/h2h3_test_r1_s0/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/provenance_by_dispatch/test06-run43939566725/h2h3_test_r1_s1/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/provenance_by_dispatch/test07-run43951333068/h2h3_test_r1_s0/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/provenance_by_dispatch/test07-run43951333068/h2h3_test_r1_s1/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/provenance_by_dispatch/test08-run43958703608/h2h3_test_r1_s0/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/provenance_by_dispatch/test08-run43958703608/h2h3_test_r1_s1/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/provenance_by_dispatch/test09-run43967192969/h2h3_test_r1_s0/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/provenance_by_dispatch/test09-run43967192969/h2h3_test_r1_s1/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/provenance_by_dispatch/test10-run43975815163/h2h3_test_r1_s0/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/data/provenance_by_dispatch/test10-run43975815163/h2h3_test_r1_s1/provenance.json`

| line | category | reason |
|---|---|---|
| 9 | infrastructure | private-repository CI URL (contains the account name); github_run_id kept |
| 10 | infrastructure | hostname of the CI runner machine |

### `package_B/evidence/agent_h2h_v3.json`

| line | category | reason |
|---|---|---|
| 18 | infrastructure | private-repository CI URL (contains the account name); run id kept |
| 25 | infrastructure | private-repository CI URL (contains the account name); run id kept |

### `package_B/evidence/agent_h2h_v3_report.json`

| line | category | reason |
|---|---|---|
| 16 | infrastructure | private-repository CI URL (contains the account name); run id kept (2 occurrences) |

### `package_B/evidence/agent_runs.json`

| line | category | reason |
|---|---|---|
| 20 | infrastructure | private-repository CI URL (contains the account name); run id kept |
| 39 | infrastructure | private-repository CI URL (contains the account name); run id kept |

### `package_B/evidence/harness_costs.json`

| line | category | reason |
|---|---|---|
| 175 | infrastructure | home-relative local path to the harness scripts |

### `package_B/evidence/orchestrator_snapshot.json`

| line | category | reason |
|---|---|---|
| 69 | unrelated | subagent role name that refers to another project (its cost share is kept) |
| 383 | infrastructure | home-relative local path to the harness scripts |

## Round 2 (2026-11-17): second, independent personal-details pass

A second reviewer re-checked every file (data included) for name forms, other people, emails, phones, postal addresses, dates of birth, identifiers, company names, machine and account traces, local time-zone offsets, and possessive or procedural references to a person's own machine, subscription, key or tooling. JSONL records were checked field by field (free text included): no personal value was found in them, and none changed. Each row below is one changed line; the removed text is never shown. Line structure is kept.

| category | log rows | distinct lines |
|---|---|---|
| personal | 58 | 58 |

By package: package_A personal 50; package_B personal 8.

By kind:

- local time-zone offset or zone name; the same instant is written in UTC: 15
- reference to a personal device; neutral wording: 13
- possessive reference to personal tooling or logs; neutral wording: 7
- reference to a personal subscription or login; neutral wording: 5
- reference to a personal provider-console project; neutral wording: 5
- possessive reference to a personal key; neutral wording: 5
- gendered pronoun; neutral wording: 3
- author name outside the single creator line: 2
- local time-zone name; neutral wording: 2
- possessive reference to a personal check; neutral wording: 1

Also changed in round 2 (not package content):

- `MANIFEST.json`: the contact address in `creator` is replaced by a personal marker; the `status` wording is neutral; 30 git author/committer dates that carried a local UTC offset are written as the same instant in UTC; sha256, bytes and redaction-row counts are regenerated for the 16 files changed here (original blobs unchanged); a `redaction_round_2` record is added.
- `README.md`: neutral status wording; the creator note no longer mentions a contact address; the round-2 checks are described.
- This log, line 162: a reason that named a personal machine is reworded.

Kept by decision (to be confirmed before release): the single creator line (README and the `creator` record of `MANIFEST.json`); role-only mentions of "the owner" as the person who approved or decided a step (no personal detail attached); the statement that list prices are not the operator's bill because the operator is on a subscription (cost method); cited authors of published work.

### Round 2 per file

#### `package_A/DECISIONS_excerpt.md`

| line | category | reason |
|---|---|---|
| 3 | personal | local time-zone name; neutral wording |
| 79 | personal | possessive reference to a personal check; neutral wording |
| 84 | personal | reference to a personal subscription or login; neutral wording |
| 92 | personal | local time-zone offset or zone name; the same instant is written in UTC |
| 95 | personal | local time-zone offset or zone name; the same instant is written in UTC |
| 101 | personal | possessive reference to personal tooling or logs; neutral wording |
| 113 | personal | reference to a personal provider-console project; neutral wording |
| 134 | personal | possessive reference to a personal key; neutral wording |
| 135 | personal | reference to a personal device; neutral wording |
| 138 | personal | possessive reference to personal tooling or logs; neutral wording |
| 208 | personal | local time-zone offset or zone name; the same instant is written in UTC |
| 230 | personal | gendered pronoun; neutral wording |
| 264 | personal | local time-zone offset or zone name; the same instant is written in UTC |
| 265 | personal | local time-zone offset or zone name; the same instant is written in UTC |
| 267 | personal | reference to a personal device; neutral wording |
| 333 | personal | local time-zone name; neutral wording |
| 430 | personal | local time-zone offset or zone name; the same instant is written in UTC |
| 602 | personal | local time-zone offset or zone name; the same instant is written in UTC |

#### `package_A/PREREG.md`

| line | category | reason |
|---|---|---|
| 8 | personal | author name outside the single creator line |
| 12 | personal | possessive reference to personal tooling or logs; neutral wording |
| 60 | personal | reference to a personal subscription or login; neutral wording |
| 200 | personal | reference to a personal device; neutral wording |
| 205 | personal | possessive reference to personal tooling or logs; neutral wording |
| 209 | personal | reference to a personal subscription or login; neutral wording |
| 253 | personal | local time-zone offset or zone name; the same instant is written in UTC |

#### `package_A/PREREG_heldout.md`

| line | category | reason |
|---|---|---|
| 28 | personal | possessive reference to personal tooling or logs; neutral wording |
| 29 | personal | reference to a personal device; neutral wording |

#### `package_A/PREREG_v2.md`

| line | category | reason |
|---|---|---|
| 21 | personal | reference to a personal provider-console project; neutral wording |
| 22 | personal | reference to a personal provider-console project; neutral wording |
| 130 | personal | gendered pronoun; neutral wording |
| 199 | personal | gendered pronoun; neutral wording |
| 204 | personal | reference to a personal device; neutral wording |
| 205 | personal | reference to a personal device; neutral wording |

#### `package_A/PREREG_v3.md`

| line | category | reason |
|---|---|---|
| 478 | personal | possessive reference to a personal key; neutral wording |
| 489 | personal | local time-zone offset or zone name; the same instant is written in UTC |

#### `package_A/versions/PREREG_1f63cd0.md`

| line | category | reason |
|---|---|---|
| 8 | personal | author name outside the single creator line |
| 12 | personal | possessive reference to personal tooling or logs; neutral wording |
| 60 | personal | reference to a personal subscription or login; neutral wording |
| 200 | personal | reference to a personal device; neutral wording |
| 205 | personal | possessive reference to personal tooling or logs; neutral wording |
| 209 | personal | reference to a personal subscription or login; neutral wording |
| 253 | personal | local time-zone offset or zone name; the same instant is written in UTC |

#### `package_A/versions/PREREG_v2_260aa00.md`

| line | category | reason |
|---|---|---|
| 21 | personal | reference to a personal provider-console project; neutral wording |
| 22 | personal | reference to a personal provider-console project; neutral wording |

#### `package_A/versions/PREREG_v3_925b525.md`

| line | category | reason |
|---|---|---|
| 478 | personal | possessive reference to a personal key; neutral wording |
| 489 | personal | local time-zone offset or zone name; the same instant is written in UTC |

#### `package_A/versions/PREREG_v3_c666e81.md`

| line | category | reason |
|---|---|---|
| 478 | personal | possessive reference to a personal key; neutral wording |
| 489 | personal | local time-zone offset or zone name; the same instant is written in UTC |

#### `package_A/versions/PREREG_v3_e859d0d.md`

| line | category | reason |
|---|---|---|
| 478 | personal | possessive reference to a personal key; neutral wording |
| 489 | personal | local time-zone offset or zone name; the same instant is written in UTC |

#### `package_B/code/agent/smoke_openai.py`

| line | category | reason |
|---|---|---|
| 3 | personal | reference to a personal device; neutral wording |

#### `package_B/code/tools/claude_usage.py`

| line | category | reason |
|---|---|---|
| 2 | personal | reference to a personal device; neutral wording |

#### `package_B/code/tools/kimi_store.py`

| line | category | reason |
|---|---|---|
| 2 | personal | reference to a personal device; neutral wording |

#### `package_B/code/tools/pull_evidence.py`

| line | category | reason |
|---|---|---|
| 2 | personal | reference to a personal device; neutral wording |
| 57 | personal | reference to a personal device; neutral wording |

#### `package_B/code/tools/requirements-read.txt`

| line | category | reason |
|---|---|---|
| 1 | personal | reference to a personal device; neutral wording |

#### `package_B/evidence/heldout.json`

| line | category | reason |
|---|---|---|
| 36 | personal | local time-zone offset or zone name; the same instant is written in UTC |
| 659 | personal | local time-zone offset or zone name; the same instant is written in UTC |

## Round 3 (2026-11-17): indirect and inferential identifiers

A third pass looked for details that identify or profile the author indirectly: calendar dates that followed the local time zone rather than UTC, session, message and request identifiers, CI run ids, commit SHAs and branch names, spending, plan and usage-habit details, the remaining role wording about the owner, names of private harness scripts and files, provider-account artefacts, free text inside the JSONL records, and file metadata. Each row below is one change; the removed text is never shown. Line structure is kept.

| category | log rows | distinct lines |
|---|---|---|
| infrastructure | 33 | 33 |
| personal | 69 | 67 |
| **all** | 102 | 100 |

2 lines carry two changes (one row each).

By package: package_A infrastructure 4; package_A personal 67; package_B infrastructure 29; package_B personal 2.

By kind:

- role reference to the owner; reworded to 'the author', as in the paper: 53
- name of a private harness script or file; marker: 28
- local calendar date; the UTC date is written instead: 10
- local calendar date in a section heading or entry rule; written in UTC: 6
- reference to self-hosted model infrastructure; neutral wording: 5

Also changed in round 3 (not package content):

- `MANIFEST.json`: one timestamp-source note used local time-of-day wording and now says "the same day"; sha256, bytes and a `redaction_rows_round_3` count are regenerated for the 18 changed files (original blobs unchanged); a `redaction_round_3` record is added.
- `README.md`: the round-3 checks are described.
- This log: line 7 and this section.

Checked in round 3 and found clean (no change): no UUID, provider message or request id, session id or provider project or organisation id occurs in any file; every timestamp in the JSONL records is UTC; the JSONL free text outside public HotpotQA questions, answers and model replies is limited to one HTTP 401 message and one spend-guard note, with no path, host or account detail; the package files carry no embedded metadata. Items that need the author's decision (absolute timestamps, CI run ids and SHAs, the subscription remark, role names in the cost table, the v1 local model alias) are listed in the review notes, not changed here.

### Round 3 per file

#### `package_A/DECISIONS_excerpt.md`

| line | category | reason |
|---|---|---|
| 3 | personal | local calendar date in a section heading or entry rule; written in UTC |
| 6 | personal | local calendar date in a section heading or entry rule; written in UTC |
| 79 | personal | local calendar date; the UTC date is written instead |
| 111 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 112 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 115 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 139 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 163 | personal | local calendar date; the UTC date is written instead |
| 177 | personal | local calendar date in a section heading or entry rule; written in UTC |
| 197 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 207 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 207 | personal | local calendar date; the UTC date is written instead |
| 229 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 230 | personal | local calendar date; the UTC date is written instead |
| 230 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 235 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 242 | personal | local calendar date; the UTC date is written instead |
| 375 | personal | local calendar date; the UTC date is written instead |
| 398 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 413 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 415 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 421 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 426 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 613 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 654 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 656 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 671 | personal | role reference to the owner; reworded to 'the author', as in the paper |

#### `package_A/PREREG.md`

| line | category | reason |
|---|---|---|
| 73 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 206 | infrastructure | name of a private harness script or file; marker |
| 207 | infrastructure | name of a private harness script or file; marker |
| 268 | personal | role reference to the owner; reworded to 'the author', as in the paper |

#### `package_A/PREREG_v2.md`

| line | category | reason |
|---|---|---|
| 120 | personal | local calendar date in a section heading or entry rule; written in UTC |
| 130 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 131 | personal | local calendar date; the UTC date is written instead |
| 135 | personal | local calendar date; the UTC date is written instead |
| 140 | personal | local calendar date; the UTC date is written instead |
| 143 | personal | local calendar date; the UTC date is written instead |
| 198 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 199 | personal | role reference to the owner; reworded to 'the author', as in the paper |

#### `package_A/PREREG_v3.md`

| line | category | reason |
|---|---|---|
| 244 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 443 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 477 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 485 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 525 | personal | local calendar date in a section heading or entry rule; written in UTC |
| 544 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 545 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 550 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 559 | personal | role reference to the owner; reworded to 'the author', as in the paper |

#### `package_A/versions/PREREG_1f63cd0.md`

| line | category | reason |
|---|---|---|
| 73 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 206 | infrastructure | name of a private harness script or file; marker |
| 207 | infrastructure | name of a private harness script or file; marker |

#### `package_A/versions/PREREG_v3_2b29b46.md`

| line | category | reason |
|---|---|---|
| 244 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 443 | personal | role reference to the owner; reworded to 'the author', as in the paper |

#### `package_A/versions/PREREG_v3_925b525.md`

| line | category | reason |
|---|---|---|
| 244 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 443 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 477 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 485 | personal | role reference to the owner; reworded to 'the author', as in the paper |

#### `package_A/versions/PREREG_v3_9e0121d.md`

| line | category | reason |
|---|---|---|
| 244 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 443 | personal | role reference to the owner; reworded to 'the author', as in the paper |

#### `package_A/versions/PREREG_v3_bfcde41.md`

| line | category | reason |
|---|---|---|
| 244 | personal | role reference to the owner; reworded to 'the author', as in the paper |

#### `package_A/versions/PREREG_v3_c666e81.md`

| line | category | reason |
|---|---|---|
| 244 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 443 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 477 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 485 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 525 | personal | local calendar date in a section heading or entry rule; written in UTC |
| 531 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 542 | personal | role reference to the owner; reworded to 'the author', as in the paper |

#### `package_A/versions/PREREG_v3_e859d0d.md`

| line | category | reason |
|---|---|---|
| 244 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 443 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 477 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 485 | personal | role reference to the owner; reworded to 'the author', as in the paper |

#### `package_B/code/agent/run_grid.py`

| line | category | reason |
|---|---|---|
| 120 | personal | role reference to the owner; reworded to 'the author', as in the paper |
| 129 | personal | role reference to the owner; reworded to 'the author', as in the paper |

#### `package_B/code/tools/kimi_store.py`

| line | category | reason |
|---|---|---|
| 4 | infrastructure | name of a private harness script or file; marker |
| 30 | infrastructure | reference to self-hosted model infrastructure; neutral wording |

#### `package_B/code/tools/pull_evidence.py`

| line | category | reason |
|---|---|---|
| 6 | infrastructure | name of a private harness script or file; marker |
| 7 | infrastructure | name of a private harness script or file; marker |
| 11 | infrastructure | name of a private harness script or file; marker |
| 41 | infrastructure | name of a private harness script or file; marker |
| 55 | infrastructure | name of a private harness script or file; marker |
| 65 | infrastructure | name of a private harness script or file; marker |
| 151 | infrastructure | reference to self-hosted model infrastructure; neutral wording |
| 187 | infrastructure | name of a private harness script or file; marker |
| 198 | infrastructure | reference to self-hosted model infrastructure; neutral wording |
| 498 | infrastructure | name of a private harness script or file; marker |
| 679 | infrastructure | name of a private harness script or file; marker |
| 697 | infrastructure | name of a private harness script or file; marker |
| 742 | infrastructure | name of a private harness script or file; marker |
| 745 | infrastructure | name of a private harness script or file; marker |
| 756 | infrastructure | name of a private harness script or file; marker |
| 781 | infrastructure | name of a private harness script or file; marker |
| 784 | infrastructure | name of a private harness script or file; marker |

#### `package_B/evidence/baseline_sample.json`

| line | category | reason |
|---|---|---|
| 51 | infrastructure | name of a private harness script or file; marker |
| 57 | infrastructure | name of a private harness script or file; marker |

#### `package_B/evidence/harness_costs.json`

| line | category | reason |
|---|---|---|
| 60 | infrastructure | reference to self-hosted model infrastructure; neutral wording |
| 168 | infrastructure | reference to self-hosted model infrastructure; neutral wording |
| 175 | infrastructure | name of a private harness script or file; marker |
| 189 | infrastructure | name of a private harness script or file; marker |

#### `package_B/evidence/index_ab.json`

| line | category | reason |
|---|---|---|
| 186 | infrastructure | name of a private harness script or file; marker |
| 192 | infrastructure | name of a private harness script or file; marker |

#### `package_B/evidence/orchestrator_snapshot.json`

| line | category | reason |
|---|---|---|
| 383 | infrastructure | name of a private harness script or file; marker |
| 390 | infrastructure | name of a private harness script or file; marker |

## Round 4 (2026-11-17): the author's decisions on the round-3 review list

Applied after the author reviewed the round-3 list of optional redactions. Each row below is one changed line; the removed text is never shown. Line structure is kept.

| category | log rows | distinct lines |
|---|---|---|
| infrastructure | 6 | 6 |
| personal | 3 | 3 |
| unrelated | 242 | 242 |
| **all** | 251 | 251 |

By package: package_A infrastructure 6; package_B personal 3; package_B unrelated 242.

By kind:

- producer of an evidence file left out of this release (author decision); marked stub: 242
- custom alias of a local model; neutral description: 6
- time of day of a personal harness request; truncated to the UTC date: 3

Files removed from the release in round 4 (evidence about the private harness; no figure in the paper comes from them): `package_B/evidence/harness_costs.json`, `package_B/evidence/index_ab.json`, `package_B/evidence/baseline_sample.json`. Their producer functions in `tools/pull_evidence.py` are the marked stubs listed below. The earlier rows for these three files stay in this log as a record; they no longer refer to a released file.

Not part of the release: `README_REVIEW.md`, the guide that accompanies the review copy. It is not in the repository and `make_archive.py` leaves it out.

Also changed in round 4 (not package content):

- `README.md`: the paper title is the current one; the removed files, the review-guide exclusion and the archive script are described; the round-4 checks are added.
- `MANIFEST.json`: entries of the three removed files dropped; sha256, bytes and a `redaction_rows_round_4` count regenerated for the 5 changed files; an entry for `make_archive.py`; a `redaction_round_4` record (removed and excluded files, archive note).
- `make_archive.py` (new): builds the upload zip with a fixed timestamp and no owner fields after checking every sha256 in `MANIFEST.json`. Not run yet.
- This log: line 7 and this section.

Kept as they are, by the same decision: absolute UTC timestamps, CI run ids, commit SHAs and branch names, the subscription remark, the role names in the cost table, and the free-tier note.

### Round 4 per file

#### `package_A/DECISIONS_excerpt.md`

| line | category | reason |
|---|---|---|
| 73 | infrastructure | custom alias of a local model; neutral description |
| 75 | infrastructure | custom alias of a local model; neutral description |

#### `package_A/PREREG.md`

| line | category | reason |
|---|---|---|
| 87 | infrastructure | custom alias of a local model; neutral description |
| 120 | infrastructure | custom alias of a local model; neutral description |

#### `package_A/versions/PREREG_1f63cd0.md`

| line | category | reason |
|---|---|---|
| 87 | infrastructure | custom alias of a local model; neutral description |
| 120 | infrastructure | custom alias of a local model; neutral description |

#### `package_B/code/tools/pull_evidence.py`

| line | category | reason |
|---|---|---|
| 6 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 7 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 13 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 14 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 98 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 99 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 100 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 101 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 102 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 103 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 104 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 105 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 106 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 107 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 108 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 109 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 110 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 111 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 112 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 113 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 114 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 115 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 116 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 117 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 118 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 119 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 120 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 121 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 122 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 123 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 124 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 125 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 126 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 127 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 128 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 129 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 130 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 131 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 132 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 133 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 134 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 135 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 136 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 137 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 138 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 139 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 140 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 141 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 142 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 143 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 144 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 145 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 146 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 147 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 148 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 149 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 150 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 151 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 152 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 153 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 154 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 155 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 156 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 157 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 158 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 159 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 160 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 161 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 162 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 163 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 164 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 165 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 166 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 167 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 168 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 169 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 170 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 171 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 172 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 173 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 174 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 175 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 176 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 177 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 178 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 179 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 180 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 181 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 182 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 183 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 184 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 185 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 186 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 187 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 188 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 189 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 190 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 191 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 192 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 193 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 194 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 195 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 196 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 197 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 198 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 199 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 200 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 201 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 202 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 203 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 204 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 205 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 206 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 207 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 208 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 209 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 210 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 211 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 212 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 213 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 214 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 215 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 216 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 217 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 218 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 219 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 220 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 221 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 222 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 223 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 224 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 678 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 679 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 680 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 681 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 682 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 683 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 684 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 685 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 686 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 687 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 688 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 689 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 690 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 691 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 692 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 693 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 694 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 695 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 696 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 697 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 698 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 699 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 700 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 701 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 702 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 703 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 704 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 705 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 706 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 707 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 708 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 709 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 710 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 711 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 712 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 713 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 714 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 715 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 716 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 717 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 718 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 719 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 720 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 721 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 722 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 723 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 724 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 725 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 726 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 727 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 728 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 729 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 730 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 731 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 732 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 733 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 734 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 735 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 736 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 737 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 738 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 739 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 740 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 741 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 742 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 743 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 744 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 745 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 746 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 747 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 748 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 749 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 750 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 751 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 752 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 755 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 756 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 757 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 758 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 759 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 760 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 761 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 762 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 763 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 764 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 765 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 766 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 767 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 768 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 769 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 770 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 771 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 772 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 773 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 774 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 775 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 776 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 777 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 778 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 779 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 780 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 781 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 782 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 783 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 784 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 785 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 786 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 787 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 788 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 789 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |
| 790 | unrelated | producer of an evidence file left out of this release (author decision); marked stub |

#### `package_B/evidence/heldout.json`

| line | category | reason |
|---|---|---|
| 191 | personal | time of day of a personal harness request; truncated to the UTC date |
| 192 | personal | time of day of a personal harness request; truncated to the UTC date |
| 242 | personal | time of day of a personal harness request; truncated to the UTC date |
