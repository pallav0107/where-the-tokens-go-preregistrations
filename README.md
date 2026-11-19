# Pre-registrations: "Where the Tokens Go"

Pre-registrations for the paper *Where the Tokens Go: Auditing Cache Reads and Re-Writes in a
Multi-Agent Coding Harness* (submitted to AGENT 2027, the ICSE 2027 workshop on agents in software
engineering).

## Contents

| Path | What it is |
|---|---|
| `preregistrations/PREREG_heldout.md` | Held-out test of the context-times-requests cost model |
| `preregistrations/PREREG.md` | Controlled experiment v1 (registered, never run) |
| `preregistrations/PREREG_v2.md` | Controlled experiment v2 (superseded after one development round) |
| `preregistrations/PREREG_v3.md` | Controlled experiment v3 (the reported test), with Amendments 1-5 |
| `preregistrations/PREREG_live_check.md` | Pre-registered live check of the idle-gap / cache-lifetime mechanism on a local model server, with Amendments 1-2 |
| `preregistrations/PREREG_pause.md` | Pre-registered pause experiment: idle pauses and prompt-cache re-writes in fresh Claude Code containers on SWE-bench Verified, with Amendment 1 (published before any paid API call) |
| `preregistrations/versions/` | Each earlier registered version, at its own commit |
| `preregistrations/DECISIONS_excerpt.md` | Decision-log entries the pre-registrations cite |
| `MANIFEST.json` | Per file: original commit SHA and date, the host's server-side push time, the original git blob hash, and the sha256 of the redacted copy |
| `VERIFICATION.md` | How registration timing was checked against server-side records |
| `REDACTION_LOG.md` | Every redaction, by location and category only |

## Timing

The registrations were written in a private version-controlled repository. `MANIFEST.json` gives,
for each registration and amendment, the original commit SHA, its author date (UTC) and the first
server-side push time recorded by the host, so each can be checked against the experiment runs it
governs. An independent audit checked these timestamps before the reported test ran (see
`VERIFICATION.md`).

## Redaction

Personal details and operational details unrelated to the study were removed before publication.
Each change is marked in the text (`[REDACTED: personal]`, `[REDACTED: unrelated]` or
`[REDACTED: infrastructure]`), line structure is preserved, and `REDACTION_LOG.md` lists every
change by location and category, never its content. Scientific content (hypotheses, rules,
procedures, thresholds and timestamps) is unchanged. Because redaction changes the text, each file's
redacted sha256 differs from its original blob hash; both are listed in `MANIFEST.json`, and the
unredacted originals can be provided to programme chairs on request.

The experiment's per-request records, analysis code and evidence files will be archived with a DOI
at camera-ready, after the same redaction.

## License

The documents in this repository are licensed under CC BY 4.0 (see `LICENSE`).

## Author

Pallav Pokhriyal (Kfintech).
