# AWS-2 — Visibility & Detection

**Status:** 🔄 In progress · **Cost:** ~$5 · **MITRE:** T1562.008, T1078.004

CloudTrail → S3 + CloudWatch Logs, GuardDuty, EventBridge routing of HIGH findings to SNS,
CloudWatch metric filters (root usage), Athena hunting over CloudTrail, and Security Hub
(AFSBP + CIS v5.0.0) as the findings aggregator. Validated with a Stratus Red Team
`cloudtrail-stop` detonation and measured MTTD.

## Artifacts (as they land)
- `cloudformation/detection-pipeline.yml`
- `athena/` — 5 saved hunt queries (console logins, IAM policy changes, cross-region API,
  logging tampering, S3-made-public)
- `securityhub/` — custom insight JSON + baseline score screenshot
