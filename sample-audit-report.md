# Strategy Edge Audit — Sample Report

> **This is the public sample.** The audited strategy is our own (a pooled
> 5-symbol crypto ML strategy), so nothing here is confidential. Client
> reports follow this exact structure. Methodology version: harness-2026.06.
> Prepared: 2026-07-16. All numbers are real, from the audit of 2026-06-25.

---

## Subject

| | |
|---|---|
| Strategy | Directional ML (random forest), 5 pooled crypto pairs, 15-minute bars |
| Submitted evidence | Backtest: Sharpe **10.9**, 38 trades, max DD −7%, "passes Deflated Sharpe" |
| Data provided | Full prediction/return series: 399,850 out-of-sample bars (Jan 2024 – Apr 2026), 5 symbols |
| Claimed trial count | 1 strategy (as submitted) |
| Audit scope | Statistical validation of the historical record. No code review. Not investment advice. |

## Verdict

```
┌──────────────────────────────────────────────────────────────┐
│  REAL   ·   FRAGILE   ·  [ THIN ]  ·   OVERFIT (risk: high)  │
└──────────────────────────────────────────────────────────────┘
```

**THIN — the claimed operating point does not survive selection-bias
correction, and the honest operating point produces too few trades to
certify.** The underlying model is genuinely informative (this is not a
zero-signal strategy), but at the only confidence region where the edge is
positive after costs, the strategy fires ~1.4 trades/month across all five
symbols combined. At that frequency the observed Sharpe is statistically
indistinguishable from luck (Deflated Sharpe probability 0.345 vs the 0.95
certification bar).

## Findings

### F1 — The submitted Sharpe is a selection artifact (severity: critical)

The submitted operating point (confidence ≥ 0.80, 38 trades, Sharpe 10.9) was
chosen by scanning the *entire* out-of-sample period for the best-performing
threshold. The threshold grid shows what that scan actually did:

| confidence ≥ | trades | cost-adj Sharpe |
|---:|---:|---:|
| 0.500 | 75,588 | −0.36 |
| 0.600 | 4,955 | −0.63 |
| 0.675 | 512 | −0.13 |
| 0.700 | 255 | +0.43 |
| 0.725 | 144 | +2.23 |
| 0.750 | 97 | +4.58 |
| **0.800** | **38** | **+10.88 ← submitted** |

A Sharpe of 10.9 on 38 selected trades is not a performance estimate; it is
the maximum of ~20 correlated estimates. Sharpe ratios above ~3 on sub-100
trade samples selected post-hoc are, in our experience, artifacts in nearly
every case. Note also that the edge is negative at *every* threshold below
0.675 — the strategy has no broad edge, only a high-confidence tail.

### F2 — Honest out-of-sample protocol: the edge does not certify (critical)

We re-ran the selection honestly: the operating threshold was chosen on the
first 70% of the out-of-sample period (derivation), then applied — fixed — to
the final 30% (validation):

| | derivation (70%) | validation (30%) |
|---|---:|---:|
| chosen threshold | 0.725 (min 100 trades enforced) | 0.725 (fixed) |
| trades | 133 | **11** |
| cost-adj Sharpe | +3.39 | +2.64 |
| Deflated Sharpe prob. (n_trials=315) | — | **0.345 → FAIL** (bar: 0.95) |

The Sharpe itself held up (+2.64 — the signal is likely not pure noise). The
problem is evidential: 11 trades in 8 months × 5 symbols cannot support a
certification at any reasonable trial-count assumption. "Probably real but
unprovable at this frequency" is, for capital-allocation purposes, THIN.

### F3 — Cost resilience at the honest point: PASS (informational)

At the derivation-chosen threshold, the edge survives our full cost sweep —
it is *not* cost-fragile:

| cost tier (fee+slip per leg) | Sharpe | max DD |
|---|---:|---:|
| 10+2 bps | 12.6 | −6.9% |
| 15+5 bps (verdict tier) | 10.9 | −7.1% |
| 20+5 bps | 9.8 | −7.2% |
| 25+7.5 bps | 8.3 | −7.3% |
| 30+10 bps | 6.7 | −7.5% |

(Computed at the submitted point for illustration; the fragility profile, not
the absolute Sharpe, is the informative part — slope −0.21 Sharpe/bps.)

### F4 — Trial-count sensitivity (advisory)

The strategy was submitted as "1 trial". The threshold grid alone (F1)
embeds ~20 trials; the model family / symbol-pool / label-scheme search that
produced this configuration embeds far more. At n_trials=18 the validation
Deflated Sharpe is 0.881 — still below the bar. The verdict is not sensitive
to generous trial-count assumptions.

## What would upgrade this verdict

1. **Accumulate n ≥ 100 trades at the pre-registered threshold (0.725) in
   live/paper forward trading.** At the observed frequency this takes years
   solo; widening the instrument pool at the *same* fixed threshold is the
   honest accelerator.
2. No threshold re-picking after registration. Any re-pick resets the clock.
3. Alternatively: accept THIN as final and size the strategy as unvalidated
   (i.e., not at all, for external capital).

## Methodology (summary)

Deflated Sharpe Ratio (Bailey & López de Prado) at n_trials ∈ {18, 315};
five-tier cost stress (10+2 → 30+10 bps per leg); chronological
derivation/validation split (70/30) with threshold selection confined to the
derivation slice; overfit screens: thin-tail high Sharpe, derivation-vs-
validation delta, calendar-window restrictions, in-sample selection of any
free parameter. Full methodology document available on request.

## Limitations & disclaimer

This audit evaluates the statistical properties of the historical record as
provided. It cannot detect fabricated data (premium tier verifies against
exchange/broker statements), does not constitute investment advice or a
recommendation, and makes no forward-looking guarantee. A REAL verdict means
"the record is inconsistent with selection bias and cost blindness at the
stated confidence", not "this will make money".

---
*Strategy Edge Audit · sample report · harness-2026.06 · matt-tokarz · strategyedge@proton.me*

