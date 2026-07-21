# AWS-6 — Detection-as-Code + Measured FP Tuning

**Status:** 📋 Planned · **Cost:** ~$3

Sigma rules for AWS CloudTrail, converted to Splunk SPL (`sigma-cli`), tested with pytest
against labeled true-positive and benign event corpora, and gated by CI: a broken or
regressing rule blocks the merge.

Part B is a measured false-positive tuning journey: a deliberately noisy v1 rule tuned through
four versions (allowlist → behavioral context → threshold/aggregation), with the alert-volume
drop measured at each step and true-positive retention re-validated by detonation. Results in
[`../metrics.md`](../metrics.md).

## Layout
- `detections/` — Sigma rules (`.yml`), ATT&CK-tagged
- `splunk/` — generated SPL (build artifact, reproducible via `sigma convert`)
- `tests/` — TP + benign event corpora, pytest harness
- CI: [`.github/workflows/detections.yml`](../.github/workflows/detections.yml)
