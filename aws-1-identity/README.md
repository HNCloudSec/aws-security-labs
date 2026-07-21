# AWS-1 — Identity: IAM Guardrails + Identity Center

**Status:** ✅ Complete · **Cost:** $0 · **MITRE:** T1098, T1078

SCPs (deny-root, force-MFA via single `Deny` + `NotAction` carve-out), a developer permission
boundary with scoped `iam:PassRole`, IAM Access Analyzer, and an Identity Center
`SecurityAnalystReadOnly` Permission Set (1-hour sessions, read-only IAM with mutating verbs
explicitly denied).

## Artifacts
- `scp-deny-root.json` — organization-wide root-usage block (`ArnLike` on `aws:PrincipalArn`)
- `scp-force-mfa.json` — MFA enforcement with self-enrollment carve-out (`NotAction` + `BoolIfExists`)
- `permission-boundary-developer.json` — Allow-ceiling boundary; scoped PassRole is the deliberate residual
- `identity-center-analyst-permission-set.json` — federated read-only analyst access

## Key design decisions
- The MFA SCP uses one `Deny` with `NotAction` — a separate `Allow` cannot carve an exception
  out of a `Deny` (explicit Deny always wins), which would deadlock MFA enrollment.
- The boundary's **Allow set is the operative control**: out-of-ceiling actions are denied by
  absence. The explicit deny block is forward-looking defense-in-depth, not the mechanism.
- `BoolIfExists` on `aws:MultiFactorAuthPresent` closes the absent-key path for long-term
  access keys while leaving service-role calls unaffected.

_All account IDs sanitized to `123456789012`._
