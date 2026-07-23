# AWS-2 — Visibility & Detection

**Status:** 🔄 Base pipeline complete, extensions pending · **Cost:** ~$5 · **MITRE:** T1562.008, T1078.004

Two independent detection surfaces over a single CloudTrail source:

- **Event-driven path:** GuardDuty → EventBridge (`severity >= 7`) → SNS → email. Near-real-time.
- **Log-driven path:** CloudTrail → CloudWatch Logs → metric filter (`AccessDenied*` /
  `*UnauthorizedOperation`) → alarm → SNS → email. Carries CloudTrail delivery lag (5–15 min).

The same trail feeds both S3 (durable, tamper-evident, retrospective Athena hunting) and CloudWatch
Logs (indexed, and the only sink that can back a metric-filter alarm).

## Built & Validated ✅

| Component | Detail |
|---|---|
| CloudTrail `lab-cloudtrail` | Multi-region, log-file validation, dual sink (S3 + CW Logs) |
| CW Logs `CloudTrail/lab-logs` | 30-day retention |
| GuardDuty detector | Enabled |
| SNS `lab-security-alerts` | Email subscription confirmed; topic policy grants `sns:Publish` to `events.amazonaws.com` + `cloudwatch.amazonaws.com` |
| EventBridge `lab-guardduty-high-sev` | Pattern: `severity >= 7` → SNS |
| Metric filter + alarm `UnauthorizedAPICalls` | Sum ≥ 1 / 5 min, `treatMissingData=notBreaching` → SNS |

**Validation performed:** full GuardDuty sample-finding set fired → rule matched (`TriggeredRules`
datapoints confirmed) → email delivered. Separately, a genuine `AccessDeniedException` generated →
metric incremented → alarm transitioned → email delivered (`describe-alarm-history` →
`actionState: Succeeded`). Both notification paths confirmed by delivered message, not by
resource-created status.

See `SESSION-NOTES.md` for the four failure modes hit during this build and their root causes —
including a silent metric-name mismatch and a topic policy that revoked alarm publishing.

## Files

- `sns-policy.json` — topic policy (both service principals + account owner)
- `gd-event-pattern.json` — EventBridge pattern, severity ≥ 7
- `gd-target.json` — SNS target binding
- `metric-filter-pattern.txt` — CloudWatch Logs filter pattern
- `SESSION-NOTES.md` — build log, failures, diagnostics

## Pending

- **Step 6** — Athena workgroup + CloudTrail table + 5 saved hunt queries
- **Step 6.5** — GuardDuty suppression rules; trusted-IP vs threat-list distinction
- **Step 7** — Security Hub (AFSBP + CIS v5.0.0), baseline score, custom insight
- **Stratus detonation + MTTD measurement** — not yet performed
- **CloudFormation template** — pipeline was built imperatively via CLI; not yet ported to IaC

## Open Investigation

The metric filter surfaced unexpected `AccessDenied` volume: **~30 events per 5-minute window,
observed over a ~45-minute window on 2026-07-22**. Source not yet identified. If sustained, that's
~8k denied calls/day in a nominally idle account, and the alarm's ≥ 1 threshold is permanently
breached — a working detection with a useless threshold. First task of the AWS-6 false-positive
tuning journey: identify the repeating `eventSource` / `eventName` / `userIdentity.arn`, confirm
whether the volume is steady-state or transient, then tune against the measured baseline.
