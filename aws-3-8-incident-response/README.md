# AWS-3 / AWS-8 — Automated IR + End-to-End Incident

**Status:** 📋 Planned · **Cost:** $0 · **MITRE:** T1078, T1098, T1562.008

A Python Lambda auto-responds to GuardDuty credential findings — deactivating access keys for
`IAMUser` principals, alert-and-tag for `AssumedRole` principals (destructive role containment
is deliberately human-gated). Deployed via CloudFormation, validated end to end with a Stratus
`ec2-steal-instance-credentials` detonation, and written up as a runbook + postmortem.

## Artifacts (as they land)
- `lambda_iam_response.py` — user-vs-role branching, dry-run mode, SNS audit trail
- `cloudformation/lambda-response.yml`
- `runbook.md`, `postmortem.md`, `evidence/` — finding JSON, CloudTrail events, Lambda logs
