# Measured Results

All numbers below are **lab-measured** in this environment, with the measurement method
stated. They demonstrate methodology, not production impact — production work at prior
employers is kept separate and is not quantified here.

## Detection latency (AWS-8)
| Detonation (Stratus TTP) | GuardDuty finding | Time to finding | Time to alert (SNS) |
|---|---|---|---|
| _pending_ | | | |

**Method:** timestamp of `stratus detonate` vs. finding `createdAt` vs. SNS delivery time.

## Containment time (AWS-8)
| Finding type | Principal type | Auto-action | Finding → action complete |
|---|---|---|---|
| _pending_ | | | |

## False-positive tuning (AWS-6)
| Version | Change | Alerts / 48 h | Δ vs. v1 | True positives retained? |
|---|---|---|---|---|
| v1 | deliberately noisy baseline | _pending_ | — | |
| v2 | principal allowlist | | | |
| v3 | behavioral context | | | |
| v4 | threshold / aggregation | | | |

**Method:** alert counts via Athena over identical 48 h windows; true-positive retention
re-validated by re-detonating the matching Stratus TTP after each tune.

## ATT&CK coverage (AWS-9)
| Snapshot | Techniques covered (Cloud IaaS) | Gaps closed |
|---|---|---|
| _pending_ | | |

## IaC findings (AWS-7)
HIGH/CRITICAL findings caught pre-deploy and remediated: _pending_
