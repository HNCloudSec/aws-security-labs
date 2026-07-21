# AWS-7 — CSPM + IaC Security

**Status:** 📋 Planned · **Cost:** $0

Shift-left scanning (Checkov, Trivy, cfn-guard) against deliberately insecure templates,
runtime CSPM with Prowler compared against the Security Hub baseline, a Terraform port of the
AWS-3 response stack, and a hand-built boto3 CSPM scanner (`scanner/scanner.py`) covering IAM,
S3, and security-group checks. CI fails the build on HIGH/CRITICAL findings or a planted
secret (gitleaks).

Includes a Wiz-style attack-path writeup: three individually-moderate findings composed into
a critical toxic combination.

## Layout
- `terraform/` — IR stack in Terraform (Lambda, IAM, EventBridge)
- `scanner/scanner.py` — custom boto3 posture scanner (JSON output + severity summary)
- CI: [`.github/workflows/iac-scan.yml`](../.github/workflows/iac-scan.yml)
