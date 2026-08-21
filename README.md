# A/B Testing — Statistical Significance Analysis

Automated pipeline for evaluating A/B test results.
Pulls session and event data from BigQuery, validates it, computes conversion rates for four key metrics, and runs two-proportion z-tests to determine statistical significance — replacing manual checks through online calculators.

**SQL · Python · Statistics · Tableau**

## Business Context

The goal was to determine whether observed differences between test groups across four A/B experiments provide enough statistical evidence to support a decision to ship, investigate further, or collect more data — and to do it in a way that scales to any number of metrics without hardcoding each calculation separately.

This project uses a portfolio dataset to demonstrate a reproducible A/B testing workflow and statistical decision-making process.

## Key Results

| Test | Significant metrics | Interpretation |
| --- | --- | --- |
| 1 | 2 of 4 (`add_shipping_info` p=0.022, `begin_checkout` p=0.024) | Significant positive on checkout-flow steps — worth further business validation before shipping |
| 2 | 0 of 4 | Not significant anywhere — collect more data, not evidence of harm |
| 3 | 0 of 4 | Not significant anywhere — collect more data, not evidence of harm |
| 4 | 1 of 4 (`new_accounts` p=0.018) | Single significant result — investigate further given multiple comparisons risk |

Statistical result → interpretation → business action: significance means evidence of a difference, not by itself a decision to ship.

## Executive Summary

Test 1 showed the strongest signal, with 2 of 4 metrics significant — `add_shipping_info` (p=0.022) and `begin_checkout` (p=0.024) — while `add_payment_info` (p=0.604) and `new_accounts` (p=0.123) were not. Test 4 had a single significant result on `new_accounts` (p=0.018), on the largest sample size of the four tests (~105K sessions/group). Tests 2 and 3 showed no significant differences on any metric.

An important caveat applies across all four tests: each one checks 4 metrics simultaneously, which increases the chance of a significant result appearing by chance alone. A single significant metric — as in Test 4 — deserves more scrutiny before being treated as a confirmed effect.

A data-quality issue was also found and fixed during the project: the initial SQL query counted event occurrences (`COUNT(ga_session_id)`) instead of unique sessions (`COUNT(DISTINCT ga_session_id)`) for three of the four metrics, inflating the numerator by 2–3.5x for sessions with repeated events. This was caught during review, verified with a direct BigQuery query, corrected, and the full pipeline was re-run.

## Key Takeaway

The value of this pipeline is not only faster significance testing.
It is that grain, allocation, sample size, and multiple comparisons are checked before a result is trusted.

The bug found during this project — counting event rows instead of unique sessions — is a concrete example:
the output looked plausible until the underlying unit of analysis was verified.

For a business, this means fewer false-positive rollouts and an audit trail from dashboard number back to a reproducible calculation.

## Methodology Notes

- **Group labels:** the dataset documents `test_group` as 1 = A, 2 = B, without specifying which is the control/baseline version. Results are reported as Group 1 vs Group 2 throughout.
- **Sample Ratio Mismatch (SRM):** observed traffic allocation between groups is reported descriptively for each test, since the expected allocation ratio is not documented — this is not a formal SRM test, just a transparency check.
- **Multiple comparisons:** each test evaluates 4 metrics at once, which raises the risk of a false positive. Single significant results are flagged for additional scrutiny rather than treated as confirmed.
- **Grain validation:** session and event counts were checked with `COUNT(*)` vs `COUNT(DISTINCT ga_session_id)` before being used as conversion inputs — the bug described above was caught this way.

## What Was Done

Data was extracted from BigQuery (sessions, events, orders, account creation) and combined into a single long-format table. The pipeline validated data quality (missing values, duplicates, event coverage, grain), checked observed group allocation, then computed numerator / denominator / conversion rate for each of the 4 metrics × 4 tests using a configurable loop — not hardcoded per metric. A two-proportion z-test (`statsmodels.stats.proportion.proportions_ztest`, α = 0.05) was run on each valid result, with sanity checks on the output before exporting to CSV for Tableau.

## Recommendations

- **Test 1** — strongest candidate for further business validation on the shipping/checkout flow.
- **Test 4** — one significant metric on the largest sample; worth a closer look before acting.
- **Tests 2 and 3** — no evidence of an effect either way; retest or deprioritize rather than concluding the change failed.

## Tableau Dashboard

Live dashboard: [A/B Testing — Statistical Significance](https://public.tableau.com/views/ABTESTINGSTATISTICALSIGNIFICANCE/ABTESTINGSTATISTICALSIGNIFICANCE)

See `tableau/README.md` for details on what the dashboard shows and how significance is visualized.

## Notebook

Full analysis: `notebook/ab_test_significance_analysis.ipynb`

## Project Structure

```text
ab-testing-statistical-significance/
├── notebook/
│   └── ab_test_significance_analysis.ipynb
├── results/
│   └── ab_test_significance_results.csv
├── tableau/
│   ├── dashboard_screenshot.png
│   └── README.md
└── README.md
