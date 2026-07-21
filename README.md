# AWS Security Labs

Hands-on AWS security engineering portfolio: IAM guardrails, detection-as-code, automated
incident response, and MITRE ATT&CK coverage mapping — built in a live AWS Organization,
deployed as code, and validated with real attack detonations (Stratus Red Team).

**Companion project:** [Zero Trust Architecture Reference](https://zta-reference.com) — a live
10-layer AWS Zero Trust reference covering 75 services. That repo is the *architecture and
communication* artifact; this one is the *hands-on build and code* artifact.

<!-- CI badges: uncomment once the AWS-6 / AWS-7 workflows have run at least once
[![detections](https://github.com/HNCloudSec/aws-security-labs/actions/workflows/detections.yml/badge.svg)](https://github.com/HNCloudSec/aws-security-labs/actions/workflows/detections.yml)
[![iac-scan](https://github.com/HNCloudSec/aws-security-labs/actions/workflows/iac-scan.yml/badge.svg)](https://github.com/HNCloudSec/aws-security-labs/actions/workflows/iac-scan.yml)
-->

## Lab map

| Lab | Focus | Key artifacts | Status |
|---|---|---|---|
| [AWS-1](aws-1-identity/) | IAM guardrails: SCPs, permission boundaries, Identity Center | Deny-root + force-MFA SCPs, developer boundary, analyst Permission Set | ✅ Complete |
| [AWS-2](aws-2-detection/) | Detection pipeline: GuardDuty → EventBridge → SNS, Athena hunting, Security Hub | CFN template, 5 saved Athena hunt queries, Security Hub baseline | 🔄 In progress |
| [AWS-3 / AWS-8](aws-3-8-incident-response/) | Automated IR + end-to-end incident (detonate → detect → contain → report) | Python IR Lambda, CloudFormation stack, runbook, postmortem, evidence | 📋 Planned |
| [AWS-6](aws-6-detection-as-code/) | Detection-as-code: Sigma rules, pytest harness, CI gate + measured FP tuning | Sigma rule set, generated SPL, test corpus, FP v1→v4 writeup | 📋 Planned |
| [AWS-7](aws-7-cspm-iac/) | CSPM + IaC security: Checkov, Trivy, cfn-guard, Prowler, custom boto3 scanner | Terraform stack, `scanner.py`, CI workflow, attack-path writeup | 📋 Planned |
| [AWS-9](aws-9-attack-coverage/) | MITRE ATT&CK detection coverage map | Navigator layer JSON, prioritized gap analysis | 📋 Planned |

Measured results (detection latency, containment time, false-positive reduction) live in
[`metrics.md`](metrics.md). All numbers are lab-measured with the methodology documented —
nothing is estimated.

## Design principles

- **Everything as code.** Controls deploy via CloudFormation/Terraform; detections are
  versioned Sigma with tests; CI blocks broken rules and insecure IaC before merge.
- **Validated, not assumed.** Detections are proven with real Stratus Red Team detonations,
  not sample findings alone. Coverage is mapped to ATT&CK so gaps are visible.
- **Least-privilege by construction.** The IR Lambda's execution role carries explicit denies
  so the response system can't become an escalation path.
- **Sanitized.** Account IDs are placeholders (`123456789012`); `gitleaks` runs before every
  push.

## Environment

Personal AWS Organization: a management account (SCP authoring only), a lab account (all
resources), and an isolated SCP-test account. Budget alarms + Cost Anomaly Detection enabled;
every resource tagged `Project=aws-sec-lab` for one-query teardown.

## How to deploy

Each lab directory has its own README with deploy/teardown commands. Stacks are standalone;
AWS-2 (the detection pipeline) is the only prerequisite shared by AWS-3/AWS-8.
