# AWS-2 Session Notes — Base Detection Pipeline (2026-07-22)

**Scope:** Rebuilt and validated the AWS-2 base pipeline (CloudTrail → GuardDuty → EventBridge → SNS,
plus CloudTrail → CW Logs → metric filter → alarm → SNS) end to end. Extensions (Athena, GuardDuty
tuning, Security Hub) not yet started.

**Outcome:** Both detection surfaces proven with delivered email. Four distinct failure modes hit,
diagnosed, and fixed — see table.

---

## 1. What Was Built / Reused

| Component | Action | Validation |
|---|---|---|
| CloudTrail `lab-cloudtrail` | **Reused** — multi-region, log-file validation, dual sink (S3 + CW Logs) | `get-trail-status`: `IsLogging: true`, unbroken since 2026-03-24 |
| S3 `cloudtrail-logs-<ACCOUNT_ID>` | **Kept** — ~4 mo real CloudTrail | Athena source (Step 6) + AWS-6 benign corpus |
| S3 `lab-test-bucket-<ACCOUNT_ID>` | **Deleted** — empty scratch | verified empty before `rb` |
| CW Logs `CloudTrail/lab-logs` | Retention set to 30 days (was Never Expire) | `put-retention-policy` |
| GuardDuty detector | **Created** — 30-day trial clock started 2026-07-22 | `list-detectors` |
| SNS `lab-security-alerts` | Created + email sub (confirmed) | `list-subscriptions-by-topic` → real ARN |
| EventBridge `lab-guardduty-high-sev` | Created, `severity >= 7` → SNS | `TriggeredRules` = 173 datapoints |
| Metric filter + alarm `UnauthorizedAPICalls` | Created on CW Logs group → SNS | `describe-alarm-history` → `actionState: Succeeded` |

---

## 2. Failures — Cause → Fix (the actual value of this session)

| # | Symptom | Root Cause | Fix | Lesson |
|---|---|---|---|---|
| 1 | Sample GuardDuty finding fired, **no email**. First instinct: "SNS problem." | Finding was **severity 5.0**; EventBridge filter is `>= 7`. Rule never matched (`TriggeredRules` empty). Pipeline worked correctly. | Fired **full** sample set (`create-sample-findings` with no `--finding-types`); high-sev types cleared the bar with filter untouched. | `create-sample-findings` does **not** generate a type at its real-world severity. Isolate hop-by-hop before concluding. |
| 2 | Metric-filter alarm never left `OK`; metric always empty. | **Typo:** filter wrote to metric `UnauthorizedAPICal**ss**`; alarm watched `UnauthorizedAPICal**ls**`. Two different metrics, never met. **Zero errors surfaced anywhere.** | Deleted + recreated filter with matching name. | Silent-dead detection. Nothing alerts you that a filter and alarm are misaligned — verify names match explicitly. |
| 3 | Alarm reached `ALARM` (fresh timestamp), **no email**. | Topic policy set earlier contained only `events.amazonaws.com`. **`set-topic-attributes` replaces the policy wholesale** — it wiped the default statement, so `cloudwatch.amazonaws.com` had no `sns:Publish`. | Rewrote policy with both service principals + account-owner statement. Forced transition via `set-alarm-state` OK→ALARM to retest. | **An alarm in `ALARM` is not proof anyone was notified.** `describe-alarm-history` (`HistoryItemType: Action`) is the only place the failure appears. |
| 4 | `leave-organization` used as deny-trigger — didn't trip filter. | Returned **`ConstraintViolationException`**, not `AccessDenied`. Filter matches `AccessDenied*` / `*UnauthorizedOperation` only. | Switched to `organizations create-account` from member account → clean `AccessDeniedException`. | "Denied" is not one thing. Authorization failures ≠ constraint/validation failures. Filter precision matters. |

---

## 3. Key Commands (troubleshooting toolkit built here)

```
# did the EventBridge rule actually fire?
aws cloudwatch get-metric-statistics --namespace AWS/Events --metric-name TriggeredRules --dimensions Name=RuleName,Value=<RULE> --start-time <T> --end-time <T> --period 3600 --statistics Sum

# did the alarm's SNS action succeed? (the only place action failures appear)
aws cloudwatch describe-alarm-history --alarm-name <ALARM> --max-items 5

# is the filter writing to the metric the alarm reads?
aws logs describe-metric-filters --log-group-name <GROUP>

# force an alarm transition to retest notification (alarms only act on state CHANGES)
aws cloudwatch set-alarm-state --alarm-name <ALARM> --state-value OK --state-reason "retest"
aws cloudwatch set-alarm-state --alarm-name <ALARM> --state-value ALARM --state-reason "retest"
```

---

## 4. Insights Worth Saying Out Loud (interview material)

- **Why one trail, two sinks:** S3 = durable, tamper-evident (digest files), cheap retrospective
  forensics via Athena. CW Logs = indexed, near-real-time, and the **only** one that can back a
  metric-filter alarm. Different jobs, same source.
- **Latency differs by path:** event-bus detections (GuardDuty → EventBridge) are near-real-time;
  CloudTrail metric-filter detections carry 5–15 min delivery lag. Choose per how fast you must know.
- **173 rule triggers from one sample command.** GuardDuty is an **event stream, not a state
  snapshot** — finding updates re-emit as new events. This is precisely why you never wire
  GuardDuty → SNS raw in prod: put a Lambda / input transformer between them to dedupe by finding
  ID, format, and rate-limit. *The lab demonstrated the failure mode that justifies the prod design.*
- **Severity bands:** Low 1.0–3.9 · Medium 4.0–6.9 · High 7.0–8.9. `>= 7` = High only. Senior answer
  isn't one global cutoff — it's **tiered routing**: High → page, Medium → queue/ticket, Low → log-only.
- **Resource policies replace wholesale.** Applying a one-statement policy silently revoked every
  other principal on the topic. No error, no symptom, dead notification path.

---

## 5. Open Items → Next Session

| Item | Why it matters |
|---|---|
| **~30 `AccessDenied` events per 5-min window (~8k/day)** discovered in the account | Alarm at threshold ≥1 is permanently in `ALARM` = useless (textbook alert fatigue). **This is a real, measured noisy-v1 baseline** — better source for the AWS-6 Part B tuning journey than the synthetic noisy rule in the curriculum. Hunt the repeating `eventSource`/`eventName`/`userIdentity.arn`. |
| 173 sample GuardDuty findings | Synthetic; archive before Athena hunting / Security Hub or they pollute every query. |
| 2 self-inflicted `CreateAccount` denials in CloudTrail | Own test noise — don't mistake for a finding later. Legitimate benign-corpus entries. |
| GuardDuty trial started 2026-07-22 | See sequencing note — AWS-2/3/8 are the GuardDuty-dependent labs. |
